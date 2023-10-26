---
layout: default
title: 封装Spring Schedule
parent: Spring
---

# 一、背景

在项目中，对于需要周期性执行的任务，往往都要使用定时任务框架实现。

比较对常用的几种定时任务框架进行一个比较：

![](../../assets/images/Spring/attachments/封装Spring%20Schedule_image_0.png)

其中Spring Schedule开发起来简单，稍作改造也可以进行任务持久化。

# 二、初阶使用

## 2.1 Spring Boot集成Spring Schedule

### 2.1.1 添加maven依赖包

由于Spring Schedule包含在spring-boot-starter基础模块中了，所以不需要增加额外的依赖。

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2.1.2 启动类，添加启动注解

在springboot入口或者配置类中增加@EnableScheduling注解即可启用定时任务。

```
@EnableScheduling
@SpringBootApplication
public class ScheduleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ScheduleApplication.class, args);
    }
}
```

### 2.1.3 添加定时任务

我们将对Spring Schedule三种任务调度器分别举例说明。

#### 2.1.3.1 Cron表达式

类似于Linux下的Cron表达式时间定义规则。Cron表达式由6或7个空格分隔的时间字段组成，如下图：

![](../../assets/images/Spring/attachments/封装Spring%20Schedule_image_1.png)

常用表达式：

![](../../assets/images/Spring/attachments/封装Spring%20Schedule_image_2.png)

举个栗子：

添加一个work()方法，每10秒执行一次。

注意：当方法的执行时间超过任务调度频率时，调度器会在下个周期执行。

如：假设work()方法在第0秒开始执行，方法执行了12秒，那么下一次执行work()方法的时间是第20秒。

```
@Component
public class MyTask {
    @Scheduled(cron = "0/10 * * * * *")
    public void work() {
        // task execution logic
    }
}
```

#### 2.1.3.2 固定间隔任务

下一次的任务执行时间，是从方法最后一次任务执行结束时间开始计算。并以此规则开始周期性的执行任务。

举个栗子：

添加一个work()方法，每隔10秒执行一次。

例如：假设work()方法在第0秒开始执行，方法执行了12秒，那么下一次执行work()方法的时间是第22秒。

```
@Scheduled(fixedDelay = 1000*10)
public void work() {
    // task execution logic
}
```

#### 2.1.3.3 固定频率任务

按照指定频率执行任务，并以此规则开始周期性的执行调度。

举个栗子：

添加一个work()方法，每10秒执行一次。

注意：当方法的执行时间超过任务调度频率时，调度器会在当前方法执行完成后立即执行下次任务。

例如：假设work()方法在第0秒开始执行，方法执行了12秒，那么下一次执行work()方法的时间是第12秒。

```
@Scheduled(fixedRate = 1000*10)
public void work() {
    // task execution logic
}
```

## 2.2 配置TaskScheduler线程池

在实际项目中，我们一个系统可能会定义多个定时任务。那么多个定时任务之间是可以相互独立且可以并行执行的。

通过查看org.springframework.scheduling.config.ScheduledTaskRegistrar源代码，发现spring默认会创建一个单线程池。这样对于我们的多任务调度可能会是致命的，当多个任务并发（或需要在同一时间）执行时，任务调度器就会出现时间漂移，任务执行时间将不确定。

```
protected void scheduleTasks() {
    if (this.taskScheduler == null) {
        this.localExecutor = Executors.newSingleThreadScheduledExecutor();
        this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
    }
    //省略...
}
```

### 2.2.1 自定义线程池

新增一个配置类，实现SchedulingConfigurer接口。重写configureTasks方法，通过taskRegistrar设置自定义线程池。

```
@Configuration
public class ScheduleConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(taskExecutor());
    }
     
    @Bean(destroyMethod="shutdown")
    public Executor taskExecutor() {
        return Executors.newScheduledThreadPool(20);
    }
}
```

## 2.3 实际应用中的问题

### 2.3.1 Web应用中的启动和关闭问题

我们知道通过spring加载或初始化的Bean，在服务停止的时候，spring会自动卸载（销毁）。但是由于线程是JVM级别的，如果用户在Web应用中启动了一个线程，那么这个线程的生命周期并不会和Web应用保持一致。也就是说，即使Web应用停止了，这个线程依然没有结束（死亡）。

解决方法：

1. 当前对象是通过spring初始化

spring在卸载（销毁）实例时，会调用实例的destroy方法。通过实现DisposableBean接口覆盖destroy方法实现。在destroy方法中主动关闭线程。

```
@Component
public class MyTask implements DisposableBean{
    @Override
    public void destroy() throws Exception {
        //关闭线程或线程池
        ThreadPoolTaskScheduler scheduler = (ThreadPoolTaskScheduler)applicationContext.getBean("scheduler");
        scheduler.shutdown();
    }
    //省略...
}
```

1. 当前对象不是通过spring初始化（管理）

那么我们可以增加一个Servlet上下文监听器，在Servlet服务停止的时候主动关闭线程。

```
public class MyTaskListenter implements ServletContextListener{
    @Override
    public void contextDestroyed(ServletContextEvent arg0) {
        //关闭线程或线程池
    }
    //省略...
}
```

### 2.3.2 分布式部署问题

在实际项目中，我们的系统通常会做集群、分布式或灾备部署。那么定时任务就可能出现并发问题，即同一个任务在多个服务器上同时在运行。

解决方法（分布式锁）：

1. 通过数据库表锁

1. 通过缓存中间件

1. 通过Zookeeper实现

# 三、高阶使用

Spring Schedule的开发难度简单，但是它原生不支持任务持久化，为了解决这个问题，我们对Spring Schedule进行一层封装，使其支持任务持久化、动态启动/停止/重启、动态暂停/恢复、手动调用等功能。

## 3.1 ScheduleProperties

ScheduleProperties类的主要用途为配置定时任务线程池大小、线程组名称、线程前缀相关配置。

```
@ConfigurationProperties(prefix = "schedule")
public class ScheduleProperties {
    private int poolSize = 1;
    private String threadGroupName = "ScheduleGroup";
    private String threadNamePrefix = "ScheduleThread-";
    public int getPoolSize() {
        return poolSize;
    }
    public void setPoolSize(int poolSize) {
        this.poolSize = poolSize;
    }
    public String getThreadGroupName() {
        return threadGroupName;
    }
    public void setThreadGroupName(String threadGroupName) {
        this.threadGroupName = threadGroupName;
    }
    public String getThreadNamePrefix() {
        return threadNamePrefix;
    }
    public void setThreadNamePrefix(String threadNamePrefix) {
        this.threadNamePrefix = threadNamePrefix;
    }
}
```

## 3.2 ScheduleConfig

ScheduleConfig类的主要用途为配置ScheduledTaskRegistrar、初始化线程池、初始化启动任务等。

```
@EnableScheduling
@Configuration
@ConditionalOnClass(ScheduleService.class)
@EnableConfigurationProperties(ScheduleProperties.class)
public class ScheduleConfig implements SchedulingConfigurer {
    @Resource
    private ScheduleService scheduleService;
    @Resource
    private ScheduleProperties scheduleProperties;
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        // 初始化ThreadPoolTaskScheduler
        ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setPoolSize(scheduleProperties.getPoolSize());
        taskScheduler.setThreadGroupName(scheduleProperties.getThreadGroupName());
        taskScheduler.setThreadNamePrefix(scheduleProperties.getThreadNamePrefix());
        taskScheduler.initialize();
        taskRegistrar.setTaskScheduler(taskScheduler);
        // 保存ScheduledTaskRegistrar
        scheduleService.setScheduledTaskRegistrar(taskRegistrar);
        // 保存Set<ScheduledFuture<?>>
        Field field = ReflectionUtils.findField(ScheduledTaskRegistrar.class, "scheduledTasks");
        field.setAccessible(true);
        try {
            scheduleService.setScheduledFutureSet((Set<ScheduledFuture<?>>) field.get(taskRegistrar));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        // 启动Schedule任务
        scheduleService.getScheduleList().forEach(schedule -> {
            scheduleService.start(schedule.getImpl());
        });
    }
}
```

## 3.3 ScheduleRunnable

ScheduleRunnable类的主要用途为定义了定时任务的实现方法、暴露给使用者实现具体的业务代码。

```
public interface ScheduleRunnable extends Runnable {
    void execute();
    @Override
    default void run() {
        if (SpringUtils.getBean(ScheduleService.class).isEnable(this.getClass().getName())) {
            execute();
        }
    }
}
```

## 3.4 Schedule

Schedule类的主要用途为定义定时任务的属性。

```
public class Schedule {
    /**
     * 定时任务实现类
     */
    private String impl;
    /**
     * 定时任务cron表达式
     */
    private String cron;
    /**
     * 定时任务描述
     */
    private String desc;
    /**
     * 定时任务状态
     */
    private String status;
    public String getImpl() {
        return impl;
    }
    public void setImpl(String impl) {
        this.impl = impl;
    }
    public String getCron() {
        return cron;
    }
    public void setCron(String cron) {
        this.cron = cron;
    }
    public String getDesc() {
        return desc;
    }
    public void setDesc(String desc) {
        this.desc = desc;
    }
    public String getStatus() {
        return status;
    }
    public void setStatus(String status) {
        this.status = status;
    }
}
```

## 3.5 ScheduleService

ScheduleService类的主要用途为定义了定时任务启动、停止、重启、暂停、恢复、执行等接口。

```
public abstract class ScheduleService {
    private ScheduledTaskRegistrar scheduledTaskRegistrar;
    private Set<ScheduledFuture<?>> scheduledFutureSet = null;
    private Map<String, ScheduledFuture<?>> scheduledFuturesMap = new ConcurrentHashMap<>();
    public abstract List<Schedule> getScheduleList();
    public abstract Schedule getScheduleByImpl(String impl);
    public abstract boolean isEnable(String impl);
    public abstract String start(String impl);
    public abstract String stop(String impl);
    public abstract String restart(String impl);
    public abstract String pause(String impl);
    public abstract String resume(String impl);
    public abstract String execute(String impl);
    public ScheduledTaskRegistrar getScheduledTaskRegistrar() {
        return scheduledTaskRegistrar;
    }
    public void setScheduledTaskRegistrar(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        this.scheduledTaskRegistrar = scheduledTaskRegistrar;
    }
    public Set<ScheduledFuture<?>> getScheduledFutureSet() {
        return scheduledFutureSet;
    }
    public void setScheduledFutureSet(Set<ScheduledFuture<?>> scheduledFutureSet) {
        this.scheduledFutureSet = scheduledFutureSet;
    }
    public Map<String, ScheduledFuture<?>> getScheduledFuturesMap() {
        return scheduledFuturesMap;
    }
    public void setScheduledFuturesMap(Map<String, ScheduledFuture<?>> scheduledFuturesMap) {
        this.scheduledFuturesMap = scheduledFuturesMap;
    }
}
```

```
@Service
public class ScheduleServiceImpl extends ScheduleService {
    private Map<String, Schedule> cache = new HashMap<>();
    @PostConstruct
    public void init() {
        Schedule schedule1 = new Schedule();
        schedule1.setImpl("cn.blueisacat.demo.schedule.runnable.impl.ScheduleRunnableImpl1");
        schedule1.setCron("* * * * * ?");
        schedule1.setDesc("this is desc1");
        schedule1.setStatus("1");
        cache.put(schedule1.getImpl(), schedule1);
        Schedule schedule2 = new Schedule();
        schedule2.setImpl("cn.blueisacat.demo.schedule.runnable.impl.ScheduleRunnableImpl2");
        schedule2.setCron("* * * * * ?");
        schedule2.setDesc("this is desc2");
        schedule2.setStatus("0");
        cache.put(schedule2.getImpl(), schedule2);
        Schedule schedule3 = new Schedule();
        schedule3.setImpl("cn.blueisacat.demo.schedule.runnable.impl.ScheduleRunnableImpl3");
        schedule3.setCron("* * * * * ?");
        schedule3.setDesc("this is desc3");
        schedule3.setStatus("1");
        cache.put(schedule3.getImpl(), schedule3);
    }
    @Override
    public List<Schedule> getScheduleList() {
        List<Schedule> list = new ArrayList<>();
        cache.forEach((impl, schedule) -> {
            list.add(schedule);
        });
        return list;
    }
    @Override
    public Schedule getScheduleByImpl(String impl) {
        return getScheduleList().stream().filter(s -> s.getImpl().equals(impl)).findFirst().orElse(null);
    }
    @Override
    public boolean isEnable(String impl) {
        boolean isEnable = false;
        Schedule schedule = getScheduleByImpl(impl);
        if (null != schedule) {
            isEnable = schedule.getStatus().equals("1");
        }
        return isEnable;
    }
    @Override
    public String start(String impl) {
        if (!getScheduledFuturesMap().containsKey(impl)) {
            Schedule schedule = getScheduleByImpl(impl);
            if (null == schedule) {
                return "Schedule任务不存在";
            }
            try {
                Class clazz = Class.forName(schedule.getImpl());
                Runnable runnable = (Runnable) SpringUtils.getBean(clazz);
                TaskScheduler taskScheduler = getScheduledTaskRegistrar().getScheduler();
                ScheduledFuture<?> future = taskScheduler.schedule(runnable, new CronTrigger(schedule.getCron()));
                getScheduledFuturesMap().put(impl, future);
                getScheduledFutureSet().add(future);
                return "启动成功";
            } catch (Exception e) {
                e.printStackTrace();
                return "启动异常";
            }
        } else {
            return "已启动，请勿重复操作";
        }
    }
    @Override
    public String stop(String impl) {
        if (getScheduledFuturesMap().containsKey(impl)) {
            ScheduledFuture<?> future = getScheduledFuturesMap().get(impl);
            if (null != future) {
                future.cancel(true);
            }
            getScheduledFuturesMap().remove(impl);
            return "停止成功";
        } else {
            return "已停止，请勿重复操作";
        }
    }
    @Override
    public String restart(String impl) {
        stop(impl);
        return start(impl);
    }
    @Override
    public String pause(String impl) {
        getScheduleByImpl(impl).setStatus("0");
        return "暂停成功";
    }
    @Override
    public String resume(String impl) {
        getScheduleByImpl(impl).setStatus("1");
        return "恢复成功";
    }
    @Override
    public String execute(String impl) {
        try {
            Schedule schedule = getScheduleByImpl(impl);
            Class clazz = Class.forName(schedule.getImpl());
            Runnable runnable = (Runnable) SpringUtils.getBean(clazz);
            runnable.run();
            return "执行成功";
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            return "执行失败";
        }
    }
}
```

## 3.6 ScheduleController

ScheduleController类的主要用途为提供启动、停止、重启、暂停、恢复、执行的Restful接口。

```
@RestController
@RequestMapping("schedule")
public class ScheduleController {
    @Resource
    private ScheduleService scheduleService;
    @GetMapping("start")
    public String start(@RequestParam("impl") String impl) {
        return scheduleService.start(impl);
    }
    @GetMapping("stop")
    public String stop(@RequestParam("impl") String impl) {
        return scheduleService.stop(impl);
    }
    @GetMapping("restart")
    public String restart(@RequestParam("impl") String impl) {
        return scheduleService.restart(impl);
    }
    @GetMapping("pause")
    public String pause(@RequestParam("impl") String impl) {
        return scheduleService.pause(impl);
    }
    @GetMapping("resume")
    public String resume(@RequestParam("impl") String impl) {
        return scheduleService.resume(impl);
    }
    @GetMapping("execute")
    public String execute(@RequestParam("impl") String impl){
        return scheduleService.execute(impl);
    }
}
```