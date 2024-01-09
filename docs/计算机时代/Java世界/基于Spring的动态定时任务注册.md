# #基于Spring的动态定时任务注册

## ThreadPoolTaskScheduler注册

> 借用Spring的ThreadPoolTaskScheduler实现动态任务注册

```java
@Slf4j
@Component
public class ThreadPoolTaskSchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setThreadNamePrefix("ScheduleTaskPrefix_");
        taskScheduler.setErrorHandler(throwable -> log.error("{}", throwable));
        taskScheduler.setPoolSize(1); // 定时执行线程池大小. default value is 1
        taskScheduler.setAwaitTerminationSeconds(10);
        taskScheduler.setRemoveOnCancelPolicy(true);
        return taskScheduler;
    }
}
```

## 使用

```java
// expression: 1 * * * * ?
// 添加定时任务并进行调度
ScheduledFuture<?> schedule = threadPoolTaskScheduler.schedule(new Runnable() {
    @Override
    public void run() {
        System.out.println(new Date().toLocaleString() + "-->" + expression);
    }
}, new CronTrigger(expression));

// 取消定时任务, true表示如果运行则终端执行，false表示等待执行完成后取消
schedule.cancel(false);

// 判断调度是否取消
System.out.println("是否取消：" + schedule.isCancelled());

// 判断本次调度是否完成
System.out.println("是否执行完成调度：" + schedule.isDone());
```
