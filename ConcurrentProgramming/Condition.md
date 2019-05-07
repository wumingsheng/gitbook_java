# Condition

里我们提到过 Java 语言内置的管程里只有一个条件变量，而 Lock&Condition 实现的管程是支持多个条件变量的，这是二者的一个重要区别。

锁内代码在执行的时候，有时候需要满足一定的条件，当条件不满足的时候，不能执行数据的更新操作，将这些条件附加在锁上

- Condition.await() 等待，条件不满足
- Condition.signal() 条件满足


```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BlockedQueue<T>{
	
	  //可重复锁
	  final Lock lock = new ReentrantLock();
	  // 条件变量：队列不满  
	  final Condition notFull = lock.newCondition();
	  // 条件变量：队列不空  
	  final Condition notEmpty = lock.newCondition();

	  // 入队
	  void enq(T x) {
	    lock.lock();
	    try {
	      while (队列已满){
	        // 等待队列不满
	        notFull.await();
	      }  
	      // 省略入队操作...
	      // 入队后, 通知可出队
	      notEmpty.signal();
	    }finally {
	      lock.unlock();
	    }
	  }
	  // 出队
	  void deq(){
	    lock.lock();
	    try {
	      while (队列已空){
	        // 等待队列不空
	        notEmpty.await();
	      }  
	      // 省略出队操作...
	      // 出队后，通知可入队
	      notFull.signal();
	    }finally {
	      lock.unlock();
	    }  
	  }
	}


```






