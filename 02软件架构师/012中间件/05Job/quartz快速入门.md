首先上结论，quartz比较笨重，做了很多不应该它做的事，而且没有统一的管理平台，源码质量也粗糙，从选型的角度来说就是NO,推荐使用Github上国人自己研发的[xxl-job](https://github.com/xuxueli/xxl-job)

# 基础知识 #
* Job是我们自己写的逻辑代码，比如说发送一封秘密邮件。
* 触发器Trigger，配置了Job什么时候执行。
* 调度器JobSchedule，是调度框架的核心，没有它，Job和触发器没有意义。总而言之，言而总之，调度器根据触发器的配置，在特定的条件下『触发』特定的动作，比如我们设定触发器在每天凌晨1点触发我们的发邮件Job，调度器会在每天1点定时启动Job。

#示例代码

  public class QycTest {
  public static void main(String[] args) throws SchedulerException {
  // 新建一个触发器
  Trigger trigger = new SimpleTrigger("每秒执行一次共执行5次", 5 - 1, 1000);
  // 新建一个任务
  JobDetail jobDetail = new JobDetail("qyc_job_detail", QycJob.class);
  // 新建一个调度器工厂
  SchedulerFactory factory = new StdSchedulerFactory();
  // 获取一个调度器（核心对象）
  Scheduler scheduler = factory.getScheduler();
  scheduler.scheduleJob(jobDetail, trigger);
  // 开始调度job!
  scheduler.start();
  }
  }

**参考资料**
https://www.ibm.com/developerworks/cn/java/j-lo-taskschedule/