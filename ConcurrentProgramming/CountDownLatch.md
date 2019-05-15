# 线程池线程任务执行完成计数器

多个线程并行执行任务，任务之间没有关系，如果用线程的方式，我们可以使用`join()`等待线程执行完成
如果用线程池的方式呢？线程池的优点是线程的重复使用，因为频繁的创建线程有一定的资源开销
线程池可以使用任务计数器，初始化一个任务数量，每一个任务完成的时候，就将任务数量-1，`await()`会阻塞等待，知道所有的任务数量为0：即所有的任务都完成。

```java
//自定义线程池
ThreadPoolTaskExecutor poolTaskExecutor = new ThreadPoolTaskExecutor();
poolTaskExecutor.setCorePoolSize(2);
poolTaskExecutor.setMaxPoolSize(10);
poolTaskExecutor.setQueueCapacity(100);
poolTaskExecutor.setAllowCoreThreadTimeOut(true);
poolTaskExecutor.initialize();
CountDownLatch countDownLatch = new CountDownLatch(2);//任务执行完成计数器
poolTaskExecutor.execute(() -> {
    //your code...
    countDownLatch.countDown();
});
poolTaskExecutor.execute(() -> {
    //your code...
    countDownLatch.countDown();
});
countDownLatch.await();//等待任务全部完成

//自定义线程池

@Bean
public Executor echartExecutor() {
    log.info("ThreadPoolExecutor echartExecutor init ... ");
    ThreadPoolExecutor threadPoolExecutor = 
            new ThreadPoolExecutor(10, 100, 30, TimeUnit.MINUTES, new ArrayBlockingQueue<>(2),
                    Executors.defaultThreadFactory(), new AbortPolicy());
    log.info("ThreadPoolExecutor echartExecutor init completed  !!! ");
    threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
    return threadPoolExecutor;
}

```



```java
ExecutorService executorService = Executors.newFixedThreadPool(10);
CountDownLatch countDownLatch = new CountDownLatch(2);//任务执行完成计数器
executorService.execute(() -> {
    //your code ...
    countDownLatch.countDown();
});
executorService.execute(() -> {
    //your code ...
    countDownLatch.countDown();
});

countDownLatch.await();//等待任务全部完成
executorService.shutdown();
System.out.println("======执行完成了。。。");
```









