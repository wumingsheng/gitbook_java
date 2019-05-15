# 线程池使用

## 1. 线程池创建

线程池创建有三种：

1. 自定义创建线程池
2. 如果使用spring,可以使用spring自带的线程池
3. 可以使用Executors框架体系（不推介使用，because 它使用的都是无界限队列，容易发生OOM）

### 1.1 自定义创建线程池

```java
@Bean
public ThreadPoolExecutor echartExecutor() {
    log.info("ThreadPoolExecutor echartExecutor init ... ");
    ThreadPoolExecutor threadPoolExecutor = 
            new ThreadPoolExecutor(9, 27, 30, TimeUnit.MINUTES, new ArrayBlockingQueue<>(10000),
                    Executors.defaultThreadFactory(), new AbortPolicy());
    threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
    log.info("ThreadPoolExecutor echartExecutor init completed  !!! ");
    return threadPoolExecutor;
}
```
线程创建工厂也可以自己定义

```java
 static class NameTreadFactory implements ThreadFactory {

        private final AtomicInteger mThreadNum = new AtomicInteger(1);

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "my-thread-" + mThreadNum.getAndIncrement());
            System.out.println(t.getName() + " has been created");
            return t;
        }
    }
```

拒绝执行处理器也可以自己定义

```java
    public static class MyIgnorePolicy implements RejectedExecutionHandler {

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            doLog(r, e);
        }

        private void doLog(Runnable r, ThreadPoolExecutor e) {
            // 可做日志记录等
            System.err.println( r.toString() + " rejected");
//          System.out.println("completedTaskCount: " + e.getCompletedTaskCount());
        }
    }
```

* corePoolSize：表示线程池保有的最小线程数。有些项目很闲，但是也不能把人都撤了，至少要留 corePoolSize 个人坚守阵地。
* maximumPoolSize：表示线程池创建的最大线程数。当项目很忙时，就需要加人，但是也不能无限制地加，最多就加到 maximumPoolSize 个人。当项目闲下来时，就要撤人了，最多能撤到 corePoolSize 个人。
* keepAliveTime & unit：上面提到项目根据忙闲来增减人员，那在编程世界里，如何定义忙和闲呢？很简单，一个线程如果在一段时间内，都没有执行任务，说明很闲，keepAliveTime 和 unit 就是用来定义这个“一段时间”的参数。也就是说，如果一个线程空闲了keepAliveTime & unit这么久，而且线程池的线程数大于 corePoolSize ，那么这个空闲的线程就要被回收了。
* workQueue：工作队列，和上面示例代码的工作队列同义。
* threadFactory：通过这个参数你可以自定义如何创建线程，例如你可以给线程指定一个有意义的名字。
* handler：通过这个参数你可以自定义任务的拒绝策略。如果线程池中所有的线程都在忙碌，并且工作队列也满了（前提是工作队列是有界队列），那么此时提交任务，线程池就会拒绝接收。至于拒绝的策略，你可以通过 handler 这个参数来指定。ThreadPoolExecutor 已经提供了以下 4 种策略。
    * CallerRunsPolicy：提交任务的线程自己去执行该任务。
    * AbortPolicy：默认的拒绝策略，会 throws RejectedExecutionException。
    * DiscardPolicy：直接丢弃任务，没有任何异常抛出。
    * DiscardOldestPolicy：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列。

> Java 在 1.6 版本还增加了 allowCoreThreadTimeOut(boolean value) 方法，它可以让所有线程都支持超时，这意味着如果项目很闲，就会将项目组的成员都撤走。

### 1.2 spring自己的线程池


```java
@Bean(name = "threadPoolTaskExecutor")
public ThreadPoolTaskExecutor getAsyncThreadPoolTaskExecutor() {
    ThreadPoolTaskExecutor threadPool = new ThreadPoolTaskExecutor();
    threadPool.setCorePoolSize(10);
    threadPool.setMaxPoolSize(60);
    threadPool.setKeepAliveSeconds(60);
    threadPool.setQueueCapacity(10000);
    threadPool.setWaitForTasksToCompleteOnShutdown(true);
    threadPool.setAwaitTerminationSeconds(60);
    threadPool.initialize();
    return threadPool;
}
```

### 1.3 Executors工具框架体系创建线程池

```java
ExecutorService pool = Executors.newFixedThreadPool(taskSize);//创建固定线程数的线程池
newCachedThreadPool() //创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）
newSingleThreadExecutor() //创建一个单线程化的Executor
newScheduledThreadPool(int corePoolSize) //创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。
```
























