```
package org.apache.impala.planner;

...

/**
 * 创建执行计划 from an analyzed parse tree and query options.
 */
public class Planner {
  private final static Logger LOG = LoggerFactory.getLogger(Planner.class);

  // 最小化每主机资源需求，保证即使PlanNodes的估计值为零，也不会有plan node集的估计为零。
  public static final long MIN_PER_HOST_MEM_ESTIMATE_BYTES = 10 * 1024 * 1024;

  public static final ResourceProfile MIN_PER_HOST_RESOURCES =
      new ResourceProfileBuilder().setMemEstimateBytes(MIN_PER_HOST_MEM_ESTIMATE_BYTES)
      .setMinMemReservationBytes(0).build();

  private final PlannerContext ctx_;

  public Planner(AnalysisResult analysisResult, TQueryCtx queryCtx,
      EventSequence timeline) {
    ctx_ = new PlannerContext(analysisResult, queryCtx, timeline);
  }

  public TQueryCtx getQueryCtx() { return ctx_.getQueryCtx(); }
  public AnalysisContext.AnalysisResult getAnalysisResult() {
    return ctx_.getAnalysisResult();
  }

  /**
   * 返回用于执行analyzed parse tree的plan fragments 。可能返回single-node或分布式可执行计划。如果启用（通过查询参数），则为动态分区剪枝计算runtime filters。
   * Plan generation may fail and throw for the following reasons:
   * 1. Expr evaluation failed, e.g., during partition pruning.
   * 2. A certain feature is not yet implemented, e.g., physical join implementation for
   *    outer/semi joins without equi conjuncts.
   * 3. Expr substitution failed, e.g., because an expr was substituted with a type that
   *    render the containing expr semantically invalid. Analysis should have ensured
   *    that such an expr substitution during plan generation never fails. If it does,
   *    that typically means there is a bug in analysis, or a broken/missing smap.
   */
  public List<PlanFragment> createPlan() throws ImpalaException {
    SingleNodePlanner singleNodePlanner = new SingleNodePlanner(ctx_);
    DistributedPlanner distributedPlanner = new DistributedPlanner(ctx_);
    PlanNode singleNodePlan = singleNodePlanner.createSingleNodePlan();
    ctx_.getTimeline().markEvent("Single node plan created");
    List<PlanFragment> fragments = null;

    checkForSmallQueryOptimization(singleNodePlan);

    // 改写 join 。Join rewrites.
    invertJoins(singleNodePlan, ctx_.isSingleNodeExec());
    singleNodePlan = useNljForSingularRowBuilds(singleNodePlan, ctx_.getRootAnalyzer());

    singleNodePlanner.validatePlan(singleNodePlan);

    if (ctx_.isSingleNodeExec()) {
      // 创建一个包含整个single-node plan tree的fragment create one fragment containing the entire single-node plan tree
      fragments = Lists.newArrayList(new PlanFragment(
          ctx_.getNextFragmentId(), singleNodePlan, DataPartition.UNPARTITIONED));
    } else {
      // 创建分布式计划
      fragments = distributedPlanner.createPlanFragments(singleNodePlan);
    }

    // Create runtime filters.
    PlanFragment rootFragment = fragments.get(fragments.size() - 1);
    if (ctx_.getQueryOptions().getRuntime_filter_mode() != TRuntimeFilterMode.OFF) {
      RuntimeFilterGenerator.generateRuntimeFilters(ctx_, rootFragment.getPlanRoot());
      ctx_.getTimeline().markEvent("Runtime filters computed");
    }

    rootFragment.verifyTree();
    ExprSubstitutionMap rootNodeSmap = rootFragment.getPlanRoot().getOutputSmap();
    List<Expr> resultExprs = null;
    if (ctx_.isInsertOrCtas()) {
      InsertStmt insertStmt = ctx_.getAnalysisResult().getInsertStmt();
      insertStmt.substituteResultExprs(rootNodeSmap, ctx_.getRootAnalyzer());
      if (!ctx_.isSingleNodeExec()) {
        // 基于分区健重新分区 repartition on partition keys
        rootFragment = distributedPlanner.createInsertFragment(
            rootFragment, insertStmt, ctx_.getRootAnalyzer(), fragments);
      }
      // 基于clustered/noclustered plan添加一个可选排序节点到计划中 Add optional sort node to the plan, based on clustered/noclustered plan hint.
      createPreInsertSort(insertStmt, rootFragment, ctx_.getRootAnalyzer());
      // 为根fragment 设置 会被写入表的数据池(汇聚接收器)，insert 语句 set up table sink for root fragment
      rootFragment.setSink(insertStmt.createDataSink());
      resultExprs = insertStmt.getResultExprs();
    } else {
      if (ctx_.isUpdate()) {
        // 更新的接收器 update 语句 Set up update sink for root fragment 
        rootFragment.setSink(ctx_.getAnalysisResult().getUpdateStmt().createDataSink());
      } else if (ctx_.isDelete()) {
        // 删除的接收器 delete 语句 Set up delete sink for root fragment
        rootFragment.setSink(ctx_.getAnalysisResult().getDeleteStmt().createDataSink());
      } else if (ctx_.isQuery()) {
        // 查询 Sink
        rootFragment.setSink(ctx_.getAnalysisResult().getQueryStmt().createDataSink());
      }
      QueryStmt queryStmt = ctx_.getQueryStmt();
      queryStmt.substituteResultExprs(rootNodeSmap, ctx_.getRootAnalyzer());
      resultExprs = queryStmt.getResultExprs();
    }
    rootFragment.setOutputExprs(resultExprs);

    // 禁用codegen的检查使用每个节点的行的估计值，因此必须在分布式计划中完成。The check for disabling codegen uses estimates of rows per node so must be done on the distributed plan.
    checkForDisableCodegen(rootFragment.getPlanRoot());

    if (LOG.isTraceEnabled()) {
      LOG.trace("desctbl: " + ctx_.getRootAnalyzer().getDescTbl().debugString());
      LOG.trace("resultexprs: " + Expr.debugString(rootFragment.getOutputExprs()));
      LOG.trace("finalize plan fragments");
    }
    for (PlanFragment fragment: fragments) {
      fragment.finalizeExchanges(ctx_.getRootAnalyzer());
    }

    Collections.reverse(fragments);
    ctx_.getTimeline().markEvent("Distributed plan created");

    // ColumnLineageGraph 有向图，用于用于跟踪参与查询的表/列实体之间的依赖关系,有两种依赖关系（Projection dependency，Predicate dependency）
    ColumnLineageGraph graph = ctx_.getRootAnalyzer().getColumnLineageGraph();
    if (BackendConfig.INSTANCE.getComputeLineage() || RuntimeEnv.INSTANCE.isTestEnv()) {
      // 对更新和删除语句禁用血缘关系相关功能 Lineage is disabled for UPDATE AND DELETE statements
      if (ctx_.isUpdateOrDelete()) return fragments;
      // 计算列的血缘依赖关系图 Compute the column lineage graph
      if (ctx_.isInsertOrCtas()) {
        InsertStmt insertStmt = ctx_.getAnalysisResult().getInsertStmt();
        List<Expr> exprs = new ArrayList<>();
        FeTable targetTable = insertStmt.getTargetTable();
        Preconditions.checkNotNull(targetTable);
        if (targetTable instanceof FeKuduTable) {
          if (ctx_.isInsert()) {
            // 对于Kudu表上的insert语句，我们只需要考虑mentioned in the column list 的 labels of column。
            List<String> mentionedColumns = insertStmt.getMentionedColumns();
            Preconditions.checkState(!mentionedColumns.isEmpty());
            List<String> targetColLabels = new ArrayList<>();
            String tblFullName = targetTable.getFullName();
            for (String column: mentionedColumns) {
              targetColLabels.add(tblFullName + "." + column);
            }
            graph.addTargetColumnLabels(targetColLabels);
          } else {
            graph.addTargetColumnLabels(targetTable);
          }
          exprs.addAll(resultExprs);
        } else if (targetTable instanceof FeHBaseTable) {
          graph.addTargetColumnLabels(targetTable);
          exprs.addAll(resultExprs);
        } else {
          graph.addTargetColumnLabels(targetTable);
          exprs.addAll(ctx_.getAnalysisResult().getInsertStmt().getPartitionKeyExprs());
          exprs.addAll(resultExprs.subList(0,
              targetTable.getNonClusteringColumns().size()));
        }
        graph.computeLineageGraph(exprs, ctx_.getRootAnalyzer());
      } else {
        graph.addTargetColumnLabels(ctx_.getQueryStmt().getColLabels());
        graph.computeLineageGraph(resultExprs, ctx_.getRootAnalyzer());
      }
      if (LOG.isTraceEnabled()) LOG.trace("lineage: " + graph.debugString());
      ctx_.getTimeline().markEvent("Lineage info computed");
    }

    return fragments;
  }

  /**
   * 返回一个计划列表，每个计划由它们fragment trees的根表示 Return a list of plans, each represented by the root of their fragment trees.
   */
  public List<PlanFragment> createParallelPlans() throws ImpalaException {
    Preconditions.checkState(ctx_.getQueryOptions().mt_dop > 0);
    List<PlanFragment> distrPlan = createPlan();
    Preconditions.checkNotNull(distrPlan);
    ParallelPlanner planner = new ParallelPlanner(ctx_);
    List<PlanFragment> parallelPlans = planner.createPlans(distrPlan.get(0));
    // Only use one scanner thread per scan-node instance since intra-node
    // parallelism is achieved via multiple fragment instances.
    ctx_.getQueryOptions().setNum_scanner_threads(1);
    ctx_.getTimeline().markEvent("Parallel plans created");
    return parallelPlans;
  }

  /**
  EXPLAIN stmts
   * Return combined explain string for all plan fragments.
   * Includes the estimated resource requirements from the request if set.
   * Uses a default level of EXTENDED, unless overriden by the
   * 'explain_level' query option.
   */
  public String getExplainString(List<PlanFragment> fragments,
      TQueryExecRequest request) {
    // use EXTENDED by default for all non-explain statements
    TExplainLevel explainLevel = TExplainLevel.EXTENDED;
    // 为 EXPLAIN 语句和测试使用 查询选项 use the query option for explain stmts and tests (e.g., planner tests)
    if (ctx_.getAnalysisResult().isExplainStmt() || RuntimeEnv.INSTANCE.isTestEnv()) {
      explainLevel = ctx_.getQueryOptions().getExplain_level();
    }
    return getExplainString(fragments, request, explainLevel);
  }

  /**
   * Return combined explain string for all plan fragments and with an
   * explicit explain level.
   * Includes the estimated resource requirements from the request if set.
   */
  public String getExplainString(List<PlanFragment> fragments,
      TQueryExecRequest request, TExplainLevel explainLevel) {
    StringBuilder str = new StringBuilder();
    boolean hasHeader = false;

    // 只有queries, DML,等有profile Only some requests (queries, DML, etc) have a resource profile.
    if (request.isSetMax_per_host_min_mem_reservation()) {
      Preconditions.checkState(request.isSetMax_per_host_thread_reservation());
      Preconditions.checkState(request.isSetPer_host_mem_estimate());
      str.append(String.format(
          "Max Per-Host Resource Reservation: Memory=%s Threads=%d\n",
          PrintUtils.printBytes(request.getMax_per_host_min_mem_reservation()),
          request.getMax_per_host_thread_reservation()));
      str.append(String.format("Per-Host Resource Estimates: Memory=%s\n",
          PrintUtils.printBytesRoundedToMb(request.getPer_host_mem_estimate())));
      hasHeader = true;
    }
    // Warn if the planner is running in DEBUG mode.
    if (request.query_ctx.client_request.query_options.planner_testcase_mode) {
      str.append("WARNING: The planner is running in TESTCASE mode. This should only be "
          + "used by developers for debugging.\nTo disable it, do SET " +
          "PLANNER_TESTCASE_MODE=false.\n");
    }
    if (request.query_ctx.disable_codegen_hint) {
      str.append("Codegen disabled by planner\n");
    }

    // IMPALA-1983 In the case of corrupt stats, issue a warning for all queries except
    // child queries of 'compute stats'.
    if (!request.query_ctx.isSetParent_query_id() &&
        request.query_ctx.isSetTables_with_corrupt_stats() &&
        !request.query_ctx.getTables_with_corrupt_stats().isEmpty()) {
      List<String> tableNames = new ArrayList<>();
      for (TTableName tableName: request.query_ctx.getTables_with_corrupt_stats()) {
        tableNames.add(tableName.db_name + "." + tableName.table_name);
      }
      str.append(
          "WARNING: The following tables have potentially corrupt table statistics.\n" +
          "Drop and re-compute statistics to resolve this problem.\n" +
          Joiner.on(", ").join(tableNames) + "\n");
      hasHeader = true;
    }

    // Append warning about tables missing stats except for child queries of
    // 'compute stats'. The parent_query_id is only set for compute stats child queries.
    if (!request.query_ctx.isSetParent_query_id() &&
        request.query_ctx.isSetTables_missing_stats() &&
        !request.query_ctx.getTables_missing_stats().isEmpty()) {
      List<String> tableNames = new ArrayList<>();
      for (TTableName tableName: request.query_ctx.getTables_missing_stats()) {
        tableNames.add(tableName.db_name + "." + tableName.table_name);
      }
      str.append("WARNING: The following tables are missing relevant table " +
          "and/or column statistics.\n" + Joiner.on(", ").join(tableNames) + "\n");
      hasHeader = true;
    }

    if (request.query_ctx.isSetTables_missing_diskids()) {
      List<String> tableNames = new ArrayList<>();
      for (TTableName tableName: request.query_ctx.getTables_missing_diskids()) {
        tableNames.add(tableName.db_name + "." + tableName.table_name);
      }
      str.append("WARNING: The following tables have scan ranges with missing " +
          "disk id information.\n" + Joiner.on(", ").join(tableNames) + "\n");
      hasHeader = true;
    }

    if (request.query_ctx.isDisable_spilling()) {
      str.append("WARNING: Spilling is disabled for this query as a safety guard.\n" +
          "Reason: Query option disable_unsafe_spills is set, at least one table\n" +
          "is missing relevant stats, and no plan hints were given.\n");
      hasHeader = true;
    }

    if (explainLevel.ordinal() >= TExplainLevel.EXTENDED.ordinal()) {
      //  在extended explain中包含显示隐式转换的分析查询文本 In extended explain include the analyzed query text showing implicit casts
      String queryText = ctx_.getQueryStmt().toSql(SHOW_IMPLICIT_CASTS);
      String wrappedText = PrintUtils.wrapString("Analyzed query: " + queryText, 80);
      str.append(wrappedText).append("\n");
      hasHeader = true;
    }
    // 分析的查询文本必须是header中的最后一个内容。Note that the analyzed query text must be the last thing in the header.
    // This is to help tests that parse the header.

    // Add the blank line that indicates the end of the header
    if (hasHeader) str.append("\n");

    if (explainLevel.ordinal() < TExplainLevel.VERBOSE.ordinal()) {
      // Print the non-fragmented parallel plan.打印 non-fragmented 并行计划
      str.append(fragments.get(0).getExplainString(ctx_.getQueryOptions(), explainLevel));
    } else {
      // Print the fragmented parallel plan.打印 fragmented 并行计划
      for (int i = 0; i < fragments.size(); ++i) {
        PlanFragment fragment = fragments.get(i);
        str.append(fragment.getExplainString(ctx_.getQueryOptions(), explainLevel));
        if (i < fragments.size() - 1) str.append("\n");
      }
    }
    return str.toString();
  }

  /**
  计算给定计划的每主机资源配置文件，即属于每个主机查询的所有片段实例消耗的峰值资源。在“request”设置per-host resource values
   * Computes the per-host resource profile for the given plans, i.e. the peak resources
   * consumed by all fragment instances belonging to the query per host. Sets the
   * per-host resource values in 'request'.
   */
  public void computeResourceReqs(List<PlanFragment> planRoots,
      TQueryCtx queryCtx, TQueryExecRequest request) {
    Preconditions.checkState(!planRoots.isEmpty());
    Preconditions.checkNotNull(request);
    TQueryOptions queryOptions = ctx_.getRootAnalyzer().getQueryOptions();
    int mtDop = queryOptions.getMt_dop();

    // 每台机器所有 plan fragments 达到峰值的资源，假定所有plan fragments调度到所有节点上。真实的每台机器的资源需求是在调度之后计算的。
    // Peak per-host peak resources for all plan fragments, assuming that all fragments
    // are scheduled on all nodes. The actual per-host resource requirements are computed
    // after scheduling.
    ResourceProfile maxPerHostPeakResources = ResourceProfile.invalid();

    // 遍历所有 fragments 计算资源概述。由于 fragment 的概述可能依赖后继，所以自下而上计算。
    // Do a pass over all the fragments to compute resource profiles. Compute the
    // profiles bottom-up since a fragment's profile may depend on its descendants.
    List<PlanFragment> allFragments = planRoots.get(0).getNodesPostOrder();
    for (PlanFragment fragment: allFragments) {
      // Compute the per-node, per-sink and aggregate profiles for the fragment.
      fragment.computeResourceProfile(ctx_.getRootAnalyzer());

      // 不同的fragments不同步它们的Open() and Close(),所以后端不提供对于 一个fragment实例是否在其他实例获取资源之前释放资源的强保证。基于最大平行度(max degree of parallelism)所有fragment实例运行在所有后端节点上并且可以同时消耗它们峰值资源的保守估计，即，查询端峰值资源是每个fragment实例峰值资源的总和。
      // Different fragments do not synchronize their Open() and Close(), so the backend
      // does not provide strong guarantees about whether one fragment instance releases
      // resources before another acquires them. Conservatively assume that all fragment
      // instances run on all backends with max DOP, and can consume their peak resources
      // at the same time, i.e. that the query-wide peak resources is the sum of the
      // per-fragment-instance peak resources.
      maxPerHostPeakResources = maxPerHostPeakResources.sum(
          fragment.getResourceProfile().multiply(fragment.getNumInstancesPerHost(mtDop)));
    }
    planRoots.get(0).computePipelineMembership();

    Preconditions.checkState(maxPerHostPeakResources.getMemEstimateBytes() >= 0,
        maxPerHostPeakResources.getMemEstimateBytes());
    Preconditions.checkState(maxPerHostPeakResources.getMinMemReservationBytes() >= 0,
        maxPerHostPeakResources.getMinMemReservationBytes());

    maxPerHostPeakResources = MIN_PER_HOST_RESOURCES.max(maxPerHostPeakResources);

    // TODO: Remove per_host_mem_estimate from the TQueryExecRequest when AC no longer
    // needs it.
    request.setPer_host_mem_estimate(maxPerHostPeakResources.getMemEstimateBytes());
    request.setMax_per_host_min_mem_reservation(
        maxPerHostPeakResources.getMinMemReservationBytes());
    request.setMax_per_host_thread_reservation(
        maxPerHostPeakResources.getThreadReservation());
    if (LOG.isTraceEnabled()) {
      LOG.trace("Max per-host min reservation: " +
          maxPerHostPeakResources.getMinMemReservationBytes());
      LOG.trace("Max estimated per-host memory: " +
          maxPerHostPeakResources.getMemEstimateBytes());
      LOG.trace("Max estimated per-host thread reservation: " +
          maxPerHostPeakResources.getThreadReservation());
    }
  }

 // SingularRowSrcNode : 返回正在被它包含的SubplanNode处理的当前行。一个SingularRowSrcNode只能出现在SubplanNode的计划树中。SingularRowSrcNode返回它上级的substitutions map以便在SubplanNode的第二个子节点中适当的请求应用替换。
  /**
   * 遍历以 'root' 为根的 plan tree 并在以下情况时倒置joins。 Traverses the plan tree rooted at 'root' and inverts joins in the following situations:
   * 1. 如果join左侧是SingularRowSrcNode，那么翻转join 因为这样可以确保build side 只有一行
   * 2. 翻转分布式 non-equi right outer/semi joins， 因为它们没有后端支持, (any distributed left semi/outer join is ok).
   * 3. 经过评估，发现翻转后更好(see isInvertedJoinCheaper()).
   *    缺少相关统计信息时不要翻转
   * 前两条规则不依赖统计信息
   * Left Null Aware Anti Joins 左侧反关联存在NULL值由于缺少后端支持，不要翻转
   * 来自直接关联标识的查询块不翻转 Joins that originate from query blocks with a straight join hint are not inverted.
   * isLocalPlan参数标识以'root'为根的plan tree在本地单机执行，即，不发生任何数据交换
   */
  private void invertJoins(PlanNode root, boolean isLocalPlan) {
    if (root instanceof SubplanNode) {
      invertJoins(root.getChild(0), isLocalPlan);
      invertJoins(root.getChild(1), true);
    } else {
      for (PlanNode child: root.getChildren()) invertJoins(child, isLocalPlan);
    }

    if (root instanceof JoinNode) {
      JoinNode joinNode = (JoinNode) root;
      JoinOperator joinOp = joinNode.getJoinOp();

      if (!joinNode.isInvertible(isLocalPlan)) {
        // 重新计算元组id，因为它们的顺序必须与子元素的顺序相对应。
        root.computeTupleIds();
        return;
      }

      if (joinNode.getChild(0) instanceof SingularRowSrcNode) {
        // Always place a singular row src on the build side because it
        // only produces a single row.
        joinNode.invertJoin();
      } else if (!isLocalPlan && joinNode instanceof NestedLoopJoinNode &&
          (joinOp.isRightSemiJoin() || joinOp.isRightOuterJoin())) {
        // 当前联接是分布式non-equi right outer or semi join没有后端支持。反转联接以使其可执行。
        joinNode.invertJoin();
      } else if (isInvertedJoinCheaper(joinNode, isLocalPlan)) {
        joinNode.invertJoin();
      }
    }

    // 重新计算元组id，因为后端假定它们的顺序与子元素的顺序相对应。
    root.computeTupleIds();
  }

  /**
   * 翻转后joinNode更好返回 true，任何join输入缺少统计信息返回 false 
   *
   * 对于嵌套循环join，简单假设开销由build side的大小决定
   * 对于 hash joins, 开销模型有更多细节依赖于:
   * build and probe 中行数的评估: lhsCard and rhsCard
   * build and probe 中行size的评估: lhsAvgRowSize and rhsAvgRowSize
   * - est. lhs and rhs trees 执行的并行度: lhsNumNodes and rhsNumNodes. join并行度由lhs决定
   *
   * 假定:
   * join 策略是分区和行的分布是均匀的. 不到后续的执行中不会知道将选择什么样的连接策略，使用这样的假定可以简化分析。一般来说，如果一个输入足够小，broadcast  join是可行的，那么这个公式无论如何都会倾向于将该输入放在右边。
   * 处理a build row的成本是处理相同大小的a probe row 的两倍。
   * 处理行中每个字节的开销有一个固定的组件(C)(例如哈希并比较行)和一个变量组件(例如查找哈希表)。
   * 变量组件与build side的日志成比例增长，以近似于访问哈希表时命中内存中较慢级别的效果。
   *
   * 以任意时间单位度量的反转前后hash join的每个主机开销估计值为：
   *    (log_b(rhsBytes) + C) * (lhsBytes + 2 * rhsBytes) / lhsNumNodes
   *    vs.
   *    (log_b(lhsBytes) + C) * (rhsBytes + 2 * lhsBytes) / rhsNumNodes
   * where lhsBytes = lhsCard * lhsAvgRowSize and rhsBytes = rhsCard * rhsAvgRowSize
   *
   * 根据经验选择b=10和C=5，因为它似乎给出了一系列输入的合理结果。模型对参数不是特别敏感。
   *
   * 如果两边的平行度相同，那么这可以简化为比较两边的输入大小。否则，如果反转hash join会显著降低并行性，则需要在lhs和rhs字节之间存在显著差异来证明反转的合理性。
   */
  private boolean isInvertedJoinCheaper(JoinNode joinNode, boolean isLocalPlan) {
    long lhsCard = joinNode.getChild(0).getCardinality();
    long rhsCard = joinNode.getChild(1).getCardinality();
    // Need cardinality estimates to make a decision.
    if (lhsCard == -1 || rhsCard == -1) return false;
    double lhsBytes = lhsCard * joinNode.getChild(0).getAvgRowSize();
    double rhsBytes = rhsCard * joinNode.getChild(1).getAvgRowSize();
    if (joinNode instanceof NestedLoopJoinNode) {
      // For NLJ, simply try to minimize the size of the build side, since it needs to
      // be broadcast to all participating nodes.
      return lhsBytes < rhsBytes;
    }
    Preconditions.checkState(joinNode instanceof HashJoinNode);
    int lhsNumNodes = isLocalPlan ? 1 : joinNode.getChild(0).getNumNodes();
    int rhsNumNodes = isLocalPlan ? 1 : joinNode.getChild(1).getNumNodes();
    // Need parallelism to determine whether inverting a hash join is profitable.
    if (lhsNumNodes <= 0 || rhsNumNodes <= 0) return false;

    final long CONSTANT_COST_PER_BYTE = 5;
    // Add 1 to the log argument to avoid taking log of 0.
    double totalCost =
        (Math.log10(rhsBytes + 1) + CONSTANT_COST_PER_BYTE) * (lhsBytes + 2 * rhsBytes);
    double invertedTotalCost =
        (Math.log10(lhsBytes + 1) + CONSTANT_COST_PER_BYTE) * (rhsBytes + 2 * lhsBytes);
    double perNodeCost = totalCost / lhsNumNodes;
    double invertedPerNodeCost = invertedTotalCost / rhsNumNodes;
    if (LOG.isTraceEnabled()) {
      LOG.trace("isInvertedJoinCheaper() " + TupleId.printIds(joinNode.getTupleIds()));
      LOG.trace("lhsCard " + lhsCard + " lhsBytes " + lhsBytes +
          " lhsNumNodes " + lhsNumNodes);
      LOG.trace("rhsCard " + rhsCard + " rhsBytes " + rhsBytes +
          " rhsNumNodes " + rhsNumNodes);
      LOG.trace("cost " + perNodeCost + " invCost " + invertedPerNodeCost);
      LOG.trace("INVERT? " + (invertedPerNodeCost < perNodeCost));
    }
    return invertedPerNodeCost < perNodeCost;
  }

  /**
   * Converts hash joins to nested-loop joins if the right-side is a SingularRowSrcNode.
   * Does not convert Null Aware Anti Joins because we only support that join op with
   * a hash join.
   * Throws if JoinNode.init() fails on the new nested-loop join node.
   */
  private PlanNode useNljForSingularRowBuilds(PlanNode root, Analyzer analyzer)
      throws ImpalaException {
    for (int i = 0; i < root.getChildren().size(); ++i) {
      root.setChild(i, useNljForSingularRowBuilds(root.getChild(i), analyzer));
    }
    if (!(root instanceof JoinNode)) return root;
    if (root instanceof NestedLoopJoinNode) return root;
    if (!(root.getChild(1) instanceof SingularRowSrcNode)) return root;
    JoinNode joinNode = (JoinNode) root;
    if (joinNode.getJoinOp().isNullAwareLeftAntiJoin()) {
      Preconditions.checkState(joinNode instanceof HashJoinNode);
      return root;
    }
    List<Expr> otherJoinConjuncts = Lists.newArrayList(joinNode.getOtherJoinConjuncts());
    otherJoinConjuncts.addAll(joinNode.getEqJoinConjuncts());
    JoinNode newJoinNode = new NestedLoopJoinNode(joinNode.getChild(0),
        joinNode.getChild(1), joinNode.isStraightJoin(),
        joinNode.getDistributionModeHint(), joinNode.getJoinOp(), otherJoinConjuncts);
    newJoinNode.getConjuncts().addAll(joinNode.getConjuncts());
    newJoinNode.setId(joinNode.getId());
    newJoinNode.init(analyzer);
    return newJoinNode;
  }

  private void checkForSmallQueryOptimization(PlanNode singleNodePlan) {
    MaxRowsProcessedVisitor visitor = new MaxRowsProcessedVisitor();
    singleNodePlan.accept(visitor);
    // TODO: IMPALA-3335: support the optimization for plans with joins.
    if (!visitor.valid() || visitor.foundJoinNode()) return;
    // This optimization executes the plan on a single node so the threshold must
    // be based on the total number of rows processed.
    long maxRowsProcessed = visitor.getMaxRowsProcessed();
    int threshold = ctx_.getQueryOptions().exec_single_node_rows_threshold;
    if (maxRowsProcessed < threshold) {
      // Execute on a single node and disable codegen for small results
      ctx_.getQueryOptions().setNum_nodes(1);
      ctx_.getQueryCtx().disable_codegen_hint = true;
      if (maxRowsProcessed < ctx_.getQueryOptions().batch_size ||
          maxRowsProcessed < 1024 && ctx_.getQueryOptions().batch_size == 0) {
        // Only one scanner thread for small queries
        ctx_.getQueryOptions().setNum_scanner_threads(1);
      }
      // disable runtime filters
      ctx_.getQueryOptions().setRuntime_filter_mode(TRuntimeFilterMode.OFF);
    }
  }

  private void checkForDisableCodegen(PlanNode distributedPlan) {
    MaxRowsProcessedVisitor visitor = new MaxRowsProcessedVisitor();
    distributedPlan.accept(visitor);
    if (!visitor.valid()) return;
    // This heuristic threshold tries to determine if the per-node codegen time will
    // reduce per-node execution time enough to justify the cost of codegen. Per-node
    // execution time is correlated with the number of rows flowing through the plan.
    if (visitor.getMaxRowsProcessedPerNode()
        < ctx_.getQueryOptions().getDisable_codegen_rows_threshold()) {
      ctx_.getQueryCtx().disable_codegen_hint = true;
    }
  }

  /**
   * Insert a sort node on top of the plan, depending on the clustered/noclustered
   * plan hint and on the 'sort.columns' table property. If clustering is enabled in
   * insertStmt or additional columns are specified in the 'sort.columns' table property,
   * then the ordering columns will start with the clustering columns (key columns for
   * Kudu tables), so that partitions can be written sequentially in the table sink. Any
   * additional non-clustering columns specified by the 'sort.columns' property will be
   * added to the ordering columns and after any clustering columns. If no clustering is
   * requested and the table does not contain columns in the 'sort.columns' property, then
   * no sort node will be added to the plan.
   */
  public void createPreInsertSort(InsertStmt insertStmt, PlanFragment inputFragment,
       Analyzer analyzer) throws ImpalaException {
    List<Expr> orderingExprs = new ArrayList<>();

    boolean partialSort = false;
    if (insertStmt.getTargetTable() instanceof FeKuduTable) {
      // Always sort if the 'clustered' hint is present. Otherwise, don't sort if either
      // the 'noclustered' hint is present, or this is a single node exec, or if the
      // target table is unpartitioned.
      if (insertStmt.hasClusteredHint() || (!insertStmt.hasNoClusteredHint()
          && !ctx_.isSingleNodeExec() && !insertStmt.getPartitionKeyExprs().isEmpty())) {
        orderingExprs.add(
            KuduUtil.createPartitionExpr(insertStmt, ctx_.getRootAnalyzer()));
        orderingExprs.addAll(insertStmt.getPrimaryKeyExprs());
        partialSort = true;
      }
    } else if (insertStmt.requiresClustering()) {
      orderingExprs.addAll(insertStmt.getPartitionKeyExprs());
    }
    orderingExprs.addAll(insertStmt.getSortExprs());
    // Ignore constants for the sake of clustering.
    Expr.removeConstants(orderingExprs);

    if (orderingExprs.isEmpty()) return;

    // Build sortinfo to sort by the ordering exprs.
    List<Boolean> isAscOrder = Collections.nCopies(orderingExprs.size(), true);
    List<Boolean> nullsFirstParams = Collections.nCopies(orderingExprs.size(), false);
    SortInfo sortInfo = new SortInfo(orderingExprs, isAscOrder, nullsFirstParams);
    sortInfo.createSortTupleInfo(insertStmt.getResultExprs(), analyzer);
    sortInfo.getSortTupleDescriptor().materializeSlots();
    insertStmt.substituteResultExprs(sortInfo.getOutputSmap(), analyzer);

    PlanNode node = null;
    if (partialSort) {
      node = SortNode.createPartialSortNode(
          ctx_.getNextNodeId(), inputFragment.getPlanRoot(), sortInfo);
    } else {
      node = SortNode.createTotalSortNode(
          ctx_.getNextNodeId(), inputFragment.getPlanRoot(), sortInfo, 0);
    }
    node.init(analyzer);

    inputFragment.setPlanRoot(node);
  }
}
```
