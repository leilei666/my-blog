---
title: springboot定时任务执行
date: 2019-11-29 16:26:16
tags:
---

## 一.定时任务实现的几种方式：

1.Timer：这是java自带的java.util.Timer类，这个类允许你调度一个java.util.TimerTask任务。使用这种方式可以让你的程序按照某一个频度执行，但不能在指定时间运行。一般用的较少。
2.ScheduledExecutorService：也jdk自带的一个类；是基于线程池设计的定时任务类,每个调度任务都会分配到线程池中的一个线程去执行,也就是说,任务是并发执行,互不影响。
3.Spring Task：Spring3.0以后自带的task，可以将它看成一个轻量级的Quartz，而且使用起来比Quartz简单许多。
4.Quartz：这是一个功能比较强大的的调度器，可以让你的程序在指定时间执行，也可以按照某一个频度执行，配置起来稍显复杂。

## 二.使用Spring Task

1.创建任务类

     1 @Slf4j
     2 @Component
     3 public class ScheduledService {
     4     @Scheduled(cron = "0/5 * * * * *")
     5     public void scheduled(){
     6         log.info("=====>>>>>使用cron  {}",System.currentTimeMillis());
     7     }
     8     @Scheduled(fixedRate = 5000)
     9     public void scheduled1() {
    10         log.info("=====>>>>>使用fixedRate{}", System.currentTimeMillis());
    11     }
    12     @Scheduled(fixedDelay = 5000)
    13     public void scheduled2() {
    14         log.info("=====>>>>>fixedDelay{}",System.currentTimeMillis());
    15     }
    16 }

2.在主类上使用@EnableScheduling注解开启对定时任务的支持，然后启动项目创建任务类

     1 import org.springframework.boot.SpringApplication;
     2 import org.springframework.boot.autoconfigure.SpringBootApplication;
     3 import org.springframework.scheduling.annotation.EnableScheduling;
     4 
     5 @SpringBootApplication
     6 @EnableScheduling
     7 public class SpringBootScheduledApplication {
     8 
     9     public static void main(String[] args) {
    10         SpringApplication.run(SpringBootScheduledApplication.class, args);
    11     }
    
3.多线程执行

在传统的Spring项目中，我们可以在xml配置文件添加task的配置，而在SpringBoot项目中一般使用config配置类的方式添加配置，所以新建一个AsyncConfig类
   
     1 @Configuration
     2 @EnableAsync
     3 public class AsyncConfig {
     4 /*
     5 此处成员变量应该使用@Value从配置中读取
     6 */
     7 private int corePoolSize = 10;
     8 private int maxPoolSize = 200;
     9 private int queueCapacity = 10;
    10 @Bean
    11 public Executor taskExecutor() {
    12 ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    13 executor.setCorePoolSize(corePoolSize);
    14 executor.setMaxPoolSize(maxPoolSize);
    15 executor.setQueueCapacity(queueCapacity);
    16 executor.initialize();
    17 return executor;
    18 }
    19 }
    
@Configuration：表明该类是一个配置类 

@EnableAsync：开启异步事件的支持

然后在定时任务的类或者方法上添加@Async 。最后重启项目，每一个任务都是在不同的线程中

4.执行时间的配置

在上面的定时任务中，我们在方法上使用@Scheduled注解来设置任务的执行时间，并且使用三种属性配置方式：

    fixedRate：定义一个按一定频率执行的定时任务
    fixedDelay：定义一个按一定频率执行的定时任务，与上面不同的是，改属性可以配合initialDelay， 定义该任务延迟执行时间。
    cron：通过表达式来配置任务执行时间
    
5.cron表达式详解

一个cron表达式有至少6个（也可能7个）有空格分隔的时间元素。按顺序依次为：

    秒（0~59）
    分钟（0~59）
    3 小时（0~23）
    4 天（0~31）
    5 月（0~11）
    6 星期（1~7 1=SUN 或 SUN，MON，TUE，WED，THU，FRI，SAT）
    年份（1970－2099）

其中每个元素可以是一个值(如6),一个连续区间(9-12),一个间隔时间(8-18/4)(/表示每隔4小时),一个列表(1,3,5),通配符。由于”月份中的日期”和”星期中的日期”这两个元素互斥的,必须要对其中一个设置。配置实例：

    每隔5秒执行一次：/5 * ?
    每隔1分钟执行一次：0 /1 ?
    0 0 10,14,16 ? 每天上午10点，下午2点，4点
    0 0/30 9-17 ? 朝九晚五工作时间内每半小时
    0 0 12 ? * WED 表示每个星期三中午12点
    “0 0 12 ?” 每天中午12点触发
    “0 15 10 ? “ 每天上午10:15触发
    “0 15 10 ?” 每天上午10:15触发
    “0 15 10 ? *” 每天上午10:15触发
    “0 15 10 ? 2005” 2005年的每天上午10:15触发
    “0 14 * ?” 在每天下午2点到下午2:59期间的每1分钟触发
    “0 0/5 14 ?” 在每天下午2点到下午2:55期间的每5分钟触发
    “0 0/5 14,18 ?” 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
    “0 0-5 14 ?” 在每天下午2点到下午2:05期间的每1分钟触发
    “0 10,44 14 ? 3 WED” 每年三月的星期三的下午2:10和2:44触发
    “0 15 10 ? * MON-FRI” 周一至周五的上午10:15触发
    “0 15 10 15 * ?” 每月15日上午10:15触发
    “0 15 10 L * ?” 每月最后一日的上午10:15触发
    “0 15 10 ? * 6L” 每月的最后一个星期五上午10:15触发
    “0 15 10 ? * 6L 2002-2005” 2002年至2005年的每月的最后一个星期五上午10:15触发
    “0 15 10 ? * 6#3” 每月的第三个星期五上午10:15触发
    
## 三.整合Quartz

1.添加依赖
如果SpringBoot版本是2.0.0以后的，则在spring-boot-starter中已经包含了quart的依赖，则可以直接使用spring-boot-starter-quartz依赖：

    1 <dependency>
    2     <groupId>org.springframework.boot</groupId>
    3     <artifactId>spring-boot-starter-quartz</artifactId>
    4 </dependency>
    
如果是1.5.9则要使用以下添加依赖：

    1 <dependency>
    2   <groupId>org.quartz-scheduler</groupId>
    3   <artifactId>quartz</artifactId>
    4   <version>2.3.0</version>
    5 </dependency>
    6 <dependency>
    7   <groupId>org.springframework</groupId>
    8   <artifactId>spring-context-support</artifactId>
    9 </dependency>
    
2.创建任务类TestQuartz，该类主要是继承了QuartzJobBean

     1 public class TestQuartz extends QuartzJobBean {
     2     /**
     3      * 执行定时任务
     4      * @param jobExecutionContext
     5      * @throws JobExecutionException
     6      */
     7     @Override
     8     protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
     9         System.out.println("quartz task "+new Date());
    10     }
    11 }
    
3.创建配置类QuartzConfig

     1 @Configuration
     2 public class QuartzConfig {
     3     @Bean
     4     public JobDetail teatQuartzDetail(){
     5         return JobBuilder.newJob(TestQuartz.class).withIdentity("testQuartz").storeDurably().build();
     6     }
     7 
     8     @Bean
     9     public Trigger testQuartzTrigger(){
    10         SimpleScheduleBuilder scheduleBuilder = SimpleScheduleBuilder.simpleSchedule()
    11                 .withIntervalInSeconds(10)  //设置时间周期单位秒
    12                 .repeatForever();
    13         return TriggerBuilder.newTrigger().forJob(teatQuartzDetail())
    14                 .withIdentity("testQuartz")
    15                 .withSchedule(scheduleBuilder)
    16                 .build();
    17     }
    18 }
    
4.启动项目
