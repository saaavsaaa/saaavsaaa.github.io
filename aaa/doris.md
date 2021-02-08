### 1 机器状况巡检
##### 1.1机器最近运行情况
通过监控看最近的机器CPU内存网络情况，截图观察是否有监控信息中断的情况，是否有资源不足的情况
平均响应时间   
##### 1.2内核参数检查
Ulimit -n是否足够
##### 1.3机器诊断报告（需要在低峰时段运行)
观察磁盘网络等有没有退化

### 2服务状态巡检
##### 2.1 center & web 状态
是否有版本过多导致查询错误
##### 2.2 FE状态
多FE版本是否接近   
checkpoint是否正常   
FE错误日志   
元数据备份   
##### 2.3 BE状态
是否core过   
关键的错误日志   
compaction是否很多   
### 3查询巡检
检查最近的慢查询
1 是否有大量消耗资源的查询   
2 是否有建表不合理导致查询慢   
3 是否有大量数据倾斜   


http://doc.dorisdb.com/
https://www.bilibili.com/video/BV13v411s7sh

1 机器状况巡检
1.1 机器最近运⾏情况
通过监控看最近的机器CPU 内存 ⽹络情况
近三天查询QPS 前两天切了⼀部分流量，但是查询依旧有⽩屏，希望能解决
今天上午打开了测试⼤屏给了⼀部分数据，开始定位原因
1.2 系统参数检查
参考DorisDB⽂档 8.3.4章节 很多参数都没有调整
Ulimit -n 65535
1.3 机器诊断报告（需要在低峰时段运⾏）
观察磁盘 ⽹络等有没有退化
swap发现被打开，主要的性能退化来⾃这⾥，估计是上次机器重启以后机器没有默认关闭swap，导
致数据频繁换出，修改swap参数以后没有出现超时，但是仍然需要压测以后再上线。
2 服务状态巡检
2.1 center & web 状态
是否有版本过多导致查询错误
swap原因排除以后基本上没有问题
2.2 FE 状态
多FE版本是否接近
checkpoint是否正常
FE错误⽇志
元数据备份 /data/doris/deploy/fe/doris-meta/ 定期备份
FE基本正常
2.3 BE状态
是否core过 -- 没有
关键的错误⽇志 -- compaction 有⼀些错误
compaction是否很多 看版本
上图是优化后截图，优化前⼤约 30-50版本，对查询有⼀定影响
cumulative_compaction_num_threads_per_disk = 4
base_compaction_num_threads_per_disk = 2
cumulative_compaction_check_interval_seconds = 2 （原来默认10s）
新增了⼀个参数
3 查询巡检
检查最近的慢查询
•是否有⼤量消耗资源的查询
•是否有建表不合理导致查询慢
•是否有⼤量数据倾斜
慢查询1
PHP
use chgshs;
set is_report_success=true;
SELECT
CASE
WHEN age < 25 THEN '0-25岁'
WHEN age >= 25
AND age < 35 THEN '25-35岁'
WHEN age >= 35
AND age < 45 THEN '35-45岁'
WHEN age >= 45
AND age < 55 THEN '45-55岁'
WHEN age >= 55
AND age < 65 THEN '55-65岁'
WHEN age >= 65
AND age < 75 THEN '65-75岁'
WHEN age >= 75 THEN '75岁以上'
END AS 年龄段,
COUNT(DISTINCT cst_id) AS ⼈数
FROM
(
SELECT
year(now()) - year(b.bry_date) AS age,
b.cst_id
FROM
ord_cst_m b
JOIN [BROADCAST] ord_ord_dtl_d a ON a.cst_id = b.cst_id
WHERE
(
a.ord_acp_dtm >= date(now())
AND a.ord_ptr_cd = '100'
AND a.ord_sts_cd <> '90'
AND year(b.bry_date) >= 1900
AND year(b.bry_date) <= year(now())
)
) t
GROUP BY
1
ORDER BY
1 DESC
http://10.0.2.170:8000/#/editor/query_e4d82f6f7c234f44-ba884cfe1f2e95be
这个查询不稳定 ，有时候要3s
所以确定了⽅向去调整compaction 数据。
事件 ：当前建模⼤量使⽤unique，如果建模能够改成duplicate模型，可以⼤⼤降低sortmerge的开
销，提升查询性能，降低compaction的压⼒。
慢查询2
SQL
select date_order_by, date_id,date_id1,date_id2, cst, tv_in, total - tv_in not_t
v_in, total
from (select date_format(b.ord_acp_dtm,'%Y-%m-%d') date_order_by,
date_format(b.ord_acp_d
tm,'%c-%e') date_id,
date_format(b.ord_acp_d
tm,'%c')+0 date_id1,
date_format(b.ord_acp_dtm,'%
e')+0 date_id2,
count(distinct b.cst_id) cst,
sum(case
when c.prd_id is not null and a.third_pgm_id is not null th
en
b.acc_sls_amt
end) tv_in,
sum(b.acc_sls_amt) total
from chgshs.ord_ord_dtl_d_ext a join [broadcast]
chgshs.ord_ord_dtl_d b on a.ord_id = b.ord_id and a.ord_seq = b.
ord_seq
left join
(select distinct prd_id
from chgshs.prd_prd_sl_chnl_d
where sl_yn = 'Y'
and mda_cd = '10') c on b.prd_id = c.prd_id
where b.ord_acp_dtm >= date_format(DATE_SUB(now(), INTERVAL 30 DAY),'%
Y-%m-%d')
and b.ord_acp_dtm < date_format(now(),'%Y-%m-%d')
and a.ord_id = b.ord_id
and a.ord_seq = b.ord_seq
and b.ord_ptr_cd = '100'
and b.ord_sts_cd <> '90'
and b.biz_unt_cd = '5007'
and a.dept_l_id <> '2000004680' --社交新零售中⼼
group by date_format(b.ord_acp_dtm,'%Y-%m-%d'),
date_format(b.ord_acp_dtm,'%c-%e'),
date_format(b.ord_acp_dtm,'%c'),
date_format(b.ord_acp_dtm,'%e')
) t1 order by date_id1,date_id2

chgshs.ord_ord_dtl_d_ext a join [broadcast] chgshs.ord_ord_dtl_d
可以不要broadcast，这两张表做了colocated join 可以直接join
4 巡检结论
1 注意⼀下基础运维，包括磁盘容量监控，参数配置，这次主要就是swap没有关闭导致的性能问题，
2 慢查询主要还是版本⽐较多，调整了⼀些参数，也可以考虑建模⽅⾯的优化
3 复杂查询需要针对性的做⼀些sql优化调整
4 上线前请做⼀下压⼒测试，确认集群的qps瓶颈以后做好容量规划再上线


- 平均响应时间超过 x s
  - 95_quantile_query_latency
  - 95_quantile_query_latency
- QPS > 
  - query_received_count > 
  - query_failed_count
- 超时查询数量 /s   > 10
- Routine load 暂停
- Faild_job   failed job / min > 2
  - xx_failed_job
- Compaction 
  - be_max_compaction_score
  - cumulative_Compaction / s 
- FE gc  每10s gc 超过2次
  - jvm_young_gc
- FE pendding job  超过10
  - fe_pending_xxx_job
- FE 连接数量 FE连接数超过100
  - fe_connection_total
- BE versions  任意一张表的版本数超过50
  - be_max_compaction_score
- FE follower 版本落后超过 50 （用户重启，升级等操作都会引起此指标变动，暂时不做了）
  - follower 的max_journal_id 与 master的差值
