 线程池的构造函数
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```


线程池的参数


**int corePoolSize** -保留在池中的线程数，即使它们是空闲的，除非设置了允许核心线程超时(allowCoreThreadTimeOut)
**int maximumPoolSize** -池中允许的最大线程数
**long keepAliveTime** -当线程数大于核心时，这是空闲线程在终止前等待新任务的最大时间。（空闲线程的存活时间）
**TimeUnit unit** - keepAliveTime参数的时间单位
**BlockingQueue<Runnable> workQueue** -用于在任务执行之前保存任务的队列。这个队列将只保存execute方法提交的可运行任务。（如果任务很多，会将多余的任务放到队列里）
**ThreadFactory threadFactory** -执行程序创建新线程时使用的工厂
**RejectedExecutionHandler handler** -当执行被阻塞时使用的处理程序，因为线程边界和队列容量已经达到（当任务队列满了之后，按照指定的拒绝策略执行任务）


工作流程

1. 线程池创建，准备好指定数量的核心线程数，准备接收任务
1. 新的任务进来，用准备好的空闲线程执行。
   1. 核心线程满了，就将再进来的任务放入阻塞队列中。空闲的核心线程就会去阻塞队列获取任务执行。
   1. 阻塞队列满了，就直接开新线程执行，最大能开到maximumPoolSize指定的数量。
   1. maximumPoolSize都执行好了，maximumPoolSize数量空闲的线程会在keepAliveTime指定的时间后自动销毁，最终保存核心线程数。
   1. 如果线程数开到了maximumPoolSize数量，还有新任务进来，就会使用handler指定的拒绝策略进行处理



3. 所有的线程创建都是由指定的threadFactory创建的





使用
```java
/**
 * 创建线程池
 */
ThreadPoolExecutor executor = new ThreadPoolExecutor(5,
        200,
        10,
        TimeUnit.SECONDS,
        new LinkedBlockingDeque<>(100000),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.AbortPolicy());

/**
 * 开启异步任务
 */
CompletableFuture.runAsync(() -> {
    System.out.println("执行异步任务，线程ID：" + Thread.currentThread().getId());
}, executor);
```
