# Semaphore对象池-令牌桶

![](/assets/20190513180207.png)

上面的例子，我们用信号量实现了一个最简单的互斥锁功能。估计你会觉得奇怪，既然有 Java SDK 里面提供了 Lock，为啥还要提供一个 Semaphore ？其实实现一个互斥锁，仅仅是 Semaphore 的部分功能，Semaphore 还有一个功能是 Lock 不容易实现的，那就是：Semaphore 可以允许多个线程访问一个临界区。

现实中还有这种需求？有的。比较常见的需求就是我们工作中遇到的各种池化资源，例如连接池、对象池、线程池等等。其中，你可能最熟悉数据库连接池，在同一时刻，一定是允许多个线程同时使用连接池的，当然，每个连接在被释放前，是不允许其他线程使用的。

其实前不久，我在工作中也遇到了一个对象池的需求。所谓对象池呢，指的是一次性创建出 N 个对象，之后所有的线程重复利用这 N 个对象，当然对象在被释放前，也是不允许其他线程使用的。对象池，可以用 List 保存实例对象，这个很简单。但关键是限流器的设计，这里的限流，指的是不允许多于 N 个线程同时进入临界区。那如何快速实现一个这样的限流器呢？这种场景，我立刻就想到了信号量的解决方案。

信号量的计数器，在上面的例子中，我们设置成了 1，这个 1 表示只允许一个线程进入临界区，但如果我们把计数器的值设置成对象池里对象的个数 N，就能完美解决对象池的限流问题了。下面就是对象池的示例代码。


```java
class ObjPool<T, R> {

 final List<T> pool;

 // 用信号量实现限流器

 final Semaphore sem;

 // 构造函数

 ObjPool(int size, T t){

 pool = new Vector<T>(){};

 for(int i=0; i<size; i++){

 pool.add(t);

 }

 sem = new Semaphore(size);

 }

 // 利用对象池的对象，调用 func

 R exec(Function<T,R> func) {

 T t = null;

 sem.acquire();

 try {

 t = pool.remove(0);

 return func.apply(t);

 } finally {

 pool.add(t);

 sem.release();

 }

 }

}

// 创建对象池

ObjPool<Long, String> pool =

 new ObjPool<Long, String>(10, 2);

// 通过对象池获取 t，之后执行

pool.exec(t -> {

 System.out.println(t);

 return t.toString();

});
```

假如我有一个user对象，有一个user对象池，里面有3个user对象，并发获取user对象，执行user的toString方法

为什么看到效果，假如我们的toString方法耗时5s

```java
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@Setter
@Getter
@AllArgsConstructor
public class User {
	
	private String username;
	
	private Integer age;

	@Override
	public String toString() {
		try {
			Thread.currentThread().sleep(5000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return "User [username=" + username + ", age=" + age + "]";
	}
	
	
	

}

```

```java

import java.util.List;
import java.util.Vector;
import java.util.concurrent.Semaphore;

public class ObjPool {
	
	final List<User> pool;
	
	// 用信号量实现限流器 final Semaphore
	final Semaphore sem;

	public ObjPool(User... users) {

		pool = new Vector<User>();
		for (User user : users) {
			pool.add(user);
		}
		System.out.println(users.length);
		sem = new Semaphore(users.length);
	}
	
	public User remove()  {
		try {
			sem.acquire();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		User user = pool.remove(0);
		return user;
		
	}
	
	public void add(User user) {
		pool.add(user);
		sem.release();
	}
	

}

```

测试

```java
package com.example.lua.demo;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Test {
	
	public static void main(String[] args) throws Exception {
		User user1 = new User("u1", 1);
		User user2 = new User("u2", 2);
		User user3 = new User("u3", 3);
		
		ObjPool objPool = new ObjPool(user1, user2, user3);
		
		
		for(int i = 0; i < 100; i++) {
			
			new Thread(()->{
				
				User user = objPool.remove();
				String string = user.toString();
				log.info(string);
				objPool.add(user);
			}).start(); 
			
			
		}
		
	
		
		
		
	}

}

```

![](/assets/20190513184034.png)


