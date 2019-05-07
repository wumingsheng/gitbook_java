# Lock

## 使用经典·典型

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

//Java SDK 里面 Lock 的使用，有一个经典的范例，...
public class Library {

	// ReentrantLock:所谓可重入锁，顾名思义，指的是线程可以重复获取同一把锁
	private final Lock rtLock = new ReentrantLock();

	int value;

	public int get() {
		// 获取锁
		rtLock.lock();
		try {
			return value;
		} finally {
			// 保证锁能释放
			rtLock.unlock();
		}
	}

	public void addOne() {

		// 获取锁
		rtLock.lock();
		try {
			value += 1;
		} finally {
			// 保证锁能释放
			rtLock.unlock();
		}

	}

}

```

## ReentrantLock：可重入锁


所谓可重入锁，顾名思义，指的是线程可以重复获取同一把锁

一个操作涉及到多个方法，多个方法涉及到线程安全问题，多个方法可以重复使用一把相同的锁，ReentrantLock就是一把可以重复获取到的锁（同一把）

### 公平锁和非公平锁

在使用 ReentrantLock 的时候，你会发现 ReentrantLock 这个类有两个构造函数，一个是无参构造函数，一个是传入 fair 参数的构造函数。fair 参数代表的是锁的公平策略，如果传入 true 就表示需要构造一个公平锁，反之则表示要构造一个非公平锁。

锁都对应着一个等待队列，如果一个线程没有获得锁，就会进入等待队列，当有线程释放锁的时候，就需要从等待队列中唤醒`notifyAll()`一个等待的线程。如果是公平锁，唤醒的策略就是谁等待的时间长，就唤醒谁，很公平；如果是非公平锁，则不提供这个公平保证，有可能等待时间短的线程反而先被唤醒。


## 用锁的最佳实践

1. 永远只在更新对象的成员变量时加锁
2. 永远只在访问可变的成员变量时加锁
3. 永远不在调用其他对象的方法时加锁




