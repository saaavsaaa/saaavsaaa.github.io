   
    去年，在接银行存管的时候重新设计了架构，使用了dubbo，因为原来的项目中有一些需要强一致性事务的功能，以及一些诸如量不特别大所以想尽量简单、不想引入外部依赖之类的原因，所以就自己实现了个简单的分布式事务。如今又由于换了存管的银行，和一些其他不方便拿出来丢人的原因，总之架构又重新做了，换了spring cloud，数据库中间件用了sharding-jdbc，所以我写的这玩意也没啥用了，于是打算写篇博客记录一下，由于代码毕竟是公司项目里的，就不方便全部公布出来了，反正很简单，有需要的同学可以自己实现一下。

    首先，实现这个功能是基于一个假设，当所有业务代码都正常执行了，那事务就可以正常提交，所以如果业务代码正常执行完成，这时候数据库连接超时了，那造成的错误，我并没有避免。所以说实现的比较简单，量特别大的业务也不是很合适，其实挺鸡肋的，专门适应我们这种量不大，但是架子不小的项目。

    实现的思路很简单，对于需要多个节点协作的业务，执行过程中的节点在业务执行完成，该节点事务提交前，将事务拦截下来，等到所有节点业务代码都执行完，统一提交所有代码。

    拦截部分的代码也很简单，创建一个类继承DataSourceTransactionManager，使用的时候使用新创建的类就好了。我重写了其中newTransactionStatus、doBegin、doCommit、doCleanupAfterCompletion方法和JdbcTransactionObjectSupport内部类。

    newTransactionStatus方法的重写用于保存事务的状态:   

-----

    @Override
    protected DefaultTransactionStatus newTransactionStatus(TransactionDefinition definition, Object transaction, boolean newTransaction,
                                                            boolean newSynchronization, boolean debug, Object suspendedResources) {
        DefaultTransactionStatus status = super.newTransactionStatus(definition, transaction, newTransaction, newSynchronization, debug, suspendedResources);
        waitStatus.put(IdCache.INSTANCE.getKey(), status);
        return status;
    }

-----

  waitStatus是一个ConcurrentHashMap。IdCache.INSTANCE.getKey()是一个全局唯一Id，在业务服务被调用时生成，服务间互相调用时，随着各子服务的调用传递。接到这个Id的服务将此Id使用ThreadLocal保存，以便在本地事务执行过程中使用，这个Id是最后业务统一提交时用于查找待提交的事务的唯一标识。在dubbo中，可以借助Filter来接受此Id：

-----

    @Activate(group = Constants.PROVIDER)
    @Component
    public class ReceiveIdFilter implements Filter {
        @Override
        public Result invoke(Invoker<?> invoker, Invocation invocation) {
            String path = invoker.getInterface().getName() + "." + invocation.getMethodName();

            Result result = null;
            try {
                if (invocation.getAttachment("transactionId") == null) {
                    ......
                } else {
                    IdCache.INSTANCE.set(invocation.getAttachment("transactionId"));
                }

                if (StringUtils.isNotEmpty(invocation.getAttachment("ownId"))) {
                    GlobalTransaction.INSTANCE.setOwnId(invocation.getAttachment("ownId"));
                }

                result = invoker.invoke(invocation);
            }
            catch (RpcException e){
                throw e;
            }

            return result;
        }
    }

-----

  doBegin方法，判断是否是新的分布式事务，如果是，生成全局Id的方法[大概是这样](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/util/IdGenerator.java)，同时设置事务状态为Start；如果不是，。

-----



-----




```markdown
```
