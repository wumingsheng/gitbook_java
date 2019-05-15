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

### 1.4 使用线程池的注意事项

1. 强烈建议使用有界队列，这也是不推荐使用`Executors`的原因（Executors使用的都是无界队列容易发生OOM）
2. 使用有界队列，任务过多时候，容易发生拒绝策略，默认的拒绝策略是：throw RuntimeException. **默认拒绝策略要慎重使用**，建议自定义自己的拒绝策略；
并且在实际工作中，自定义的拒绝策略往往和降级策略配合使用。


## 2. Future & FutureTask

### 2.1 Future

Java 通过 ThreadPoolExecutor 提供的 3 个 submit() 方法和 1 个 FutureTask 工具类来支持获得任务执行结果的需求。
下面我们先来介绍这 3 个 submit() 方法，这 3 个方法的方法签名如下。

```java
// 提交 Runnable 任务 没有返回结果
Future<?> submit(Runnable task);
// 提交 Callable 任务 有返回结果
<T> Future<T> submit(Callable<T> task);
// 提交 Runnable 任务及结果引用  
<T> Future<T> submit(Runnable task, T result);
```

1. 提交 Runnable 任务 submit(Runnable task) ：这个方法的参数是一个 Runnable 接口，Runnable 接口的 run()方法是没有返回值的，
所以 submit(Runnable task) 这个方法返回的 Future仅可以用来断言任务已经结束了，类似于 Thread.join()。
2. 提交 Callable 任务 submit(Callable<T> task)：这个方法的参数是一个 Callable 接口，它只有一个 call() 方法，并且这个方法是有返回值的，
所以这个方法返回的 Future 对象可以通过调用其 get() 方法来获取任务的执行结果。
3. result 相当于主线程和子线程之间的桥梁，通过它主子线程可以共享数据。

```java

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor.AbortPolicy;
import java.util.concurrent.TimeUnit;

public class Main {

	public static void main(String[] args) throws Exception {
		ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 10, 5, TimeUnit.MINUTES,
				new ArrayBlockingQueue<>(30), Executors.defaultThreadFactory(), new AbortPolicy());
		threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
		Data data = new Main().new Data();
		
		Future<Data> future = threadPoolExecutor.submit(new Main().new Task(data), data);
		TimeUnit.SECONDS.sleep(2);
		System.out.println(data.getName());
		System.out.println(future.get().getName());//future.get() === result

		threadPoolExecutor.shutdown();

	}
	
	class Data {
	    String name;

	    public String getName() {
	        return name;
	    }

	    public void setName(String name) {
	        this.name = name;
	    }
	}

	class Task implements Runnable {
		Data data;
		public Task(Data result) {
			this.data = result;
		}

		@Override
		public void run() {
			System.out.println("run ============");
			data.setName("222");

		}

	}
}

```

你会发现它们的返回值都是 Future 接口，Future 接口有 5 个方法，我都列在下面了，它们分别是：

- 取消任务的方法 cancel()
- 判断任务是否已取消的方法 isCancelled()
- 判断任务是否已结束的方法 isDone()
- 以及2 个获得任务执行结果的 get() 和 get(timeout, unit)，其中最后一个 get(timeout, unit) 支持超时机制。

通过 Future 接口的这 5 个方法你会发现，我们提交的任务不但能够获取任务执行结果，还可以取消任务。
不过需要注意的是：这两个 get() 方法都是阻塞式的，如果被调用的时候，任务还没有执行完，那么调用 get() 方法的线程会阻塞，直到任务执行完才会被唤醒。




### 2.2 FutureTask

下面我们再来介绍 FutureTask 工具类。前面我们提到的 Future 是一个接口，而 FutureTask 是一个实实在在的工具类，这个工具类有两个构造函数，
它们的参数和前面介绍的 submit() 方法类似，所以这里我就不再赘述了。

```java
    
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);
```

那如何使用 FutureTask 呢？其实很简单，FutureTask 实现了 Runnable 和 Future 接口，由于实现了 Runnable 接口，所以可以将 FutureTask 对象作为任务提交给 ThreadPoolExecutor 去执行，也可以直接被 Thread 执行；又因为实现了 Future 接口，所以也能用来获得任务的执行结果。下面的示例代码是将 FutureTask 对象提交给 ThreadPoolExecutor 去执行。

```java
	ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 10, 5, TimeUnit.MINUTES,
				new ArrayBlockingQueue<>(30), Executors.defaultThreadFactory(), new AbortPolicy());
		threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
		
		FutureTask<String> futureTask = new FutureTask<>(()-> "111");
		threadPoolExecutor.submit(futureTask);
		String string = futureTask.get();//这里主线程会阻塞等待副线程完成
		
		
		System.out.println(string);//111

		threadPoolExecutor.shutdown();

```


也可以使用线程去执行

```java
FutureTask<String> futureTask = new FutureTask<>(()-> "111");
new Thread(futureTask).start();
String string = futureTask.get();//这里主线程会阻塞等待副线程完成
```


### 2.3 各个子线程结果相互引用案例

思路：通过构造器把T1线程的FutureTask传给T2线程

![](/assets/20190515143646.png)

```java

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Callable;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor.AbortPolicy;
import java.util.concurrent.TimeUnit;

public class Main {

	public static void main(String[] args) throws Exception {
		ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 10, 5, TimeUnit.MINUTES,
				new ArrayBlockingQueue<>(30), Executors.defaultThreadFactory(), new AbortPolicy());
		threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
		FutureTask<String> futureTask2 = new FutureTask<>(new Main().new T2());
		FutureTask<String> futureTask1 = new FutureTask<>(new Main().new T1(futureTask2));
		threadPoolExecutor.submit(futureTask1);
		threadPoolExecutor.submit(futureTask2);
		String result = futureTask1.get();
		System.out.println(result);

		threadPoolExecutor.shutdown();

	}
	
	class T1 implements Callable<String> {
		
		FutureTask<String> t2;

		public T1(FutureTask<String> t2) {
			this.t2 = t2;
		}
		
		@Override
		public String call() throws Exception {
			System.out.println("洗水壶1分钟");
			TimeUnit.SECONDS.sleep(1);
			System.out.println("烧开水15分钟");
			TimeUnit.SECONDS.sleep(15);
			String tea = t2.get();//获取t2子线程的运行结果
			System.out.println("开始泡制" + tea);
			return "泡茶完成";
		}
		
	}
	
	class T2 implements Callable<String> {
		

		
		@Override
		public String call() throws Exception {
			System.out.println("洗茶壶1分钟");
			TimeUnit.SECONDS.sleep(1);
			System.out.println("洗茶杯2分钟");
			TimeUnit.SECONDS.sleep(2);
			
			System.out.println("拿茶叶1分钟");
			TimeUnit.SECONDS.sleep(1);
			return "龙井";
		}
		
	}
	

}

```


## 3. Future增强CompletableFuture

![](/assets/20190515145359.png)

---

```java

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor.AbortPolicy;
import java.util.concurrent.TimeUnit;

public class Main {

	public static void main(String[] args) throws Exception {
		ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 10, 5, TimeUnit.MINUTES,
				new ArrayBlockingQueue<>(30), Executors.defaultThreadFactory(), new AbortPolicy());
		threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程
		
		CompletableFuture<Void> f1 = CompletableFuture.runAsync(()->{
			System.out.println("t1--- 洗水壶");
			sleep(1, TimeUnit.SECONDS);
			System.out.println("t1---烧开水");
			sleep(15, TimeUnit.SECONDS);
		}, threadPoolExecutor);
		
		CompletableFuture<String> f2 = CompletableFuture.supplyAsync(()->{
			System.out.println("t2--- 洗茶壶");
			sleep(1, TimeUnit.SECONDS);
			System.out.println("t2---洗茶杯");
			sleep(2, TimeUnit.SECONDS);
			System.out.println("t2---拿茶叶");
			sleep(1, TimeUnit.SECONDS);
			return "龙井";
		}, threadPoolExecutor);
		
		CompletableFuture<String> f3 = f1.thenCombineAsync(f2, (__, tf) -> {
			System.out.println("t3---拿到茶叶");
			System.out.println("t3---泡茶");
			
			return "上茶" + tf;
		}, threadPoolExecutor);

		f3.join();//等待任务3执行结果
		
		threadPoolExecutor.shutdown();

	}
	
	static void sleep(int t, TimeUnit u) {

			try {
				u.sleep(t);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		
	}
	

	

}
```


### 3.1 创建CompletableFuture对象

```java
CompletableFuture.runAsync(Runnable runnable)//不带返回结果
CompletableFuture.supplyAsync(Supplier<U> supplier)//带返回结果

//下面两个是上面两个的扩展：使用自定义的线程池执行
CompletableFuture.runAsync(Runnable runnable,Executor executor)
CompletableFuture.supplyAsync(Supplier<U> supplier,Executor executor) 
```

默认情况下 CompletableFuture 会使用公共的 ForkJoinPool 线程池，这个线程池默认创建的线程数是 CPU 的核数（也可以通过 JVM option:-Djava.util.concurrent.ForkJoinPool.common.parallelism 来设置 ForkJoinPool 线程池的线程数）。如果所有 CompletableFuture 共享一个线程池，那么一旦有任务执行一些很慢的 I/O 操作，就会导致线程池中所有线程都阻塞在 I/O 操作上，从而造成线程饥饿，进而影响整个系统的性能。

**所以，强烈建议你要根据不同的业务类型创建不同的线程池，以避免互相干扰。**

创建完 CompletableFuture 对象之后，会自动地异步执行 runnable.run() 方法或者 supplier.get() 方法

对于一个异步操作，你需要关注两个问题：一个是异步操作什么时候结束，另一个是如何获取异步操作的执行结果?

因为 CompletableFuture 类实现了 Future 接口，所以这两个问题你都可以通过 Future 接口来解决。
另外，CompletableFuture 类还实现了 CompletionStage 接口，这个接口内容实在是太丰富了，在 1.8 版本里有 40 个方法，这些方法我们该如何理解呢？


### 3.2 CompletionStage


![](/assets/20190515155920.png)

1. 串行关系

    * thenApply：接收参数并且提供返回值
    * thenAccept：支持参数，不支持返回值
    * thenRun：不支持参数，也不支持返回值
    * thenCompose：和thenApply相似，方法会创建一个子流程

```java
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 10, 5, TimeUnit.MINUTES,
				new ArrayBlockingQueue<>(30), Executors.defaultThreadFactory(), new AbortPolicy());
threadPoolExecutor.prestartAllCoreThreads(); // 预启动所有核心线程



CompletableFuture<String> f2 = CompletableFuture.supplyAsync(()-> "helloworld", threadPoolExecutor)
        .thenApply(s -> s + " hellofuture")
        .thenApply(String::toUpperCase);
String string = f2.get();

System.out.println(string);//HELLOWORLD HELLOFUTURE


threadPoolExecutor.shutdown();
```

2. and聚合关系

```java

CompletionStage<R> thenCombine(other, fn);//接收参数和返回值
CompletionStage<R> thenCombineAsync(other, fn);
CompletionStage<Void> thenAcceptBoth(other, consumer);//接收参数，不接受返回值
CompletionStage<Void> thenAcceptBothAsync(other, consumer);
CompletionStage<Void> runAfterBoth(other, action);//不支持参数和返回值
CompletionStage<Void> runAfterBothAsync(other, action);

```


3. or聚合关系

```java
CompletionStage applyToEither(other, fn);
CompletionStage applyToEitherAsync(other, fn);
CompletionStage acceptEither(other, consumer);
CompletionStage acceptEitherAsync(other, consumer);
CompletionStage runAfterEither(other, action);
CompletionStage runAfterEitherAsync(other, action);

```

> fn：接收参数和返回值 consumer: 接收参数不接受返回值   action: 不接受参数和返回值

```java
CompletableFuture<String> completableFuture1 = CompletableFuture.supplyAsync(()->{
			int nextInt = new Random().nextInt(5);
			sleep(nextInt, TimeUnit.SECONDS);
			return String.valueOf(nextInt);
		});
		
		CompletableFuture<String> completableFuture2 = CompletableFuture.supplyAsync(()->{
			int nextInt = new Random().nextInt(5);
			sleep(nextInt, TimeUnit.SECONDS);
			return String.valueOf(nextInt);
		});
		CompletableFuture<String> completableFuture = completableFuture1.applyToEither(completableFuture2, s -> s);
		String string = completableFuture.get();
		System.out.println(string);
```


4. 异常处理


虽然上面我们提到的 fn、consumer、action 它们的核心方法都不允许抛出可检查异常，但是却无法限制它们抛出运行时异常，例如下面的代码，执行 7/0 就会出现除零错误这个运行时异常。非异步编程里面，我们可以使用 try{}catch{}来捕获并处理异常，那在异步编程里面，异常该如何处理呢？


```java
	
CompletableFuture<Integer> f0 = CompletableFuture.supplyAsync(()->(7/0)).thenApply(r->r*10);

System.out.println(f0.join());


```

```java
CompletionStage exceptionally(fn);

CompletionStage<R> whenComplete(consumer);

CompletionStage<R> whenCompleteAsync(consumer);

CompletionStage<R> handle(fn);

CompletionStage<R> handleAsync(fn);

```

下面的示例代码展示了如何使用 exceptionally() 方法来处理异常，exceptionally() 的使用非常类似于 try{}catch{}中的 catch{}，但是由于支持链式编程方式，所以相对更简单。既然有 try{}catch{}，那就一定还有 try{}finally{}，whenComplete() 和 handle() 系列方法就类似于 try{}finally{}中的 finally{}，无论是否发生异常都会执行 whenComplete() 中的回调函数 consumer 和 handle() 中的回调函数 fn。whenComplete() 和 handle() 的区别在于 whenComplete() 不支持返回结果，而 handle() 是支持返回结果的。

```java
CompletableFuture<Integer> f0 = CompletableFuture.supplyAsync(()->7/0))
    .thenApply(r->r*10)
    .exceptionally(e->0);

System.out.println(f0.join());
```




















