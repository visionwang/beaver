请注意：
如果任务卡住，很长时间没有结束，请先【打jstack】，然后【重启tomcat】，最后【启用job】，任务可自行恢复
为防止任务卡住不能及时发现，请估算业务执行时间，在【job编辑页】设置【timeout报警阈值】
任务提示【处理失败】可能的原因为：
任务执行时抛出异常
任务由于应用重启而中断本次执行
应用负载高导致成功执行后回发ack超时,此种情况可认为任务是执行成功的

QSchedule 我的任务  com.ctrip.microfinance.giftcard.salesforce.salesforcejob.job.UpdateConversionRateJob

负载均衡策略：random, round robin, sticky
拆分策略：不拆分，广播，按数量拆分，按参数拆分


