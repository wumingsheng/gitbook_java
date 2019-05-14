# Guave Cache


`Cache`与`ConcurrentMap`很相似，但也不完全一样。

最基本的区别是`ConcurrentMap`会一直保存所有添加的元素，直到显式地移除。相对地，Guava Cache为了限制内存占用，通常都设定为自动回收元素。

在某些场景下，尽管LoadingCache不回收元素，它也是很有用的，因为它会自动加载缓存。


通常来说，Guava Cache适用于：

* 你愿意消耗一些内存空间来提升速度。
* 你预料到某些键会被查询一次以上。
* 缓存中存放的数据总量不会超出内存容量。

> Guava Cache是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果这不符合你的需求，请尝试Memcached这类工具

> 如果你不需要Cache中的特性，使用ConcurrentHashMap有更好的内存效率

## 1. 保存数据

保存数据分两种情况：

1. 显示设置`cache.put(key, value)`
2. 自动加载
    * CacheLoader
    * Callable

### 1.1 CacheLoader

```java
//缓存接口这里是LoadingCache，LoadingCache在缓存项不存在时可以自动加载缓存
	static final LoadingCache<String, String> loadingCache = CacheBuilder
			.newBuilder()//CacheBuilder的构造函数是私有的，只能通过其静态方法newBuilder()来获得CacheBuilder的实例
			.concurrencyLevel(1)//设置并发级别为1，并发级别是指可以同时写缓存的线程数
			.expireAfterWrite(8, TimeUnit.SECONDS)//设置写缓存后8秒钟过期
			.initialCapacity(10)//设置缓存容器的初始容量为10
			.maximumSize(50)//设置缓存最大容量为1000，超过1000之后就会按照LRU最近虽少使用算法来移除缓存项
			.recordStats()//设置要统计缓存的命中率
			.removalListener(new RemovalListener<Object, Object>() {//设置缓存的移除通知
                @Override
                public void onRemoval(RemovalNotification<Object, Object> notification) {
                    System.out.println(notification.getKey() + " was removed, cause is " + notification.getCause());
                }
            })
			.build(//build方法中可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存

				new CacheLoader<String, String>() {
	
					@Override
					public String load(String key) throws Exception {
						return "===" + key + "===";
					}

			});
```

### 1.2 Callable 

所有类型的Guava Cache，不管有没有自动加载功能，都支持`get(K,Callable<V>)`方法。

这个方法返回缓存中相应的值，或者用给定的Callable运算并把结果加入到缓存中。在整个加载方法完成前，缓存项相关的可观察状态都不会更改。这个方法简便地实现了模式"如果有缓存则返回；否则运算、缓存、然后返回"。

```java
final static Cache<String, Object> cache = CacheBuilder.newBuilder()
			.maximumSize(1000).build();
	
	public static void main(String[] args) throws ExecutionException {
		
		
		Object object = cache.get("name", new Callable<Object>() {

			@Override
			public Object call() throws Exception {
				return "wms";
			}
			
		}) ;
		
		System.out.println(object);
	}
```
## 2. 缓存回收

一个残酷的现实是，我们几乎一定没有足够的内存缓存所有数据。你你必须决定：什么时候某个缓存项就不值得保留了？

1. 显示回收
  * 个别清除：Cache.invalidate(key)
  * 批量清除：Cache.invalidateAll(keys)
  * 清除所有缓存项：Cache.invalidateAll()
2. 自动回收
  * 基于容量回收
  * 定时回收
  * 基于引用回收。

### 2.1 基于容量的回收（size-based eviction）

如果要规定缓存项的数目不超过固定值，只需使用`CacheBuilder.maximumSize(long)`。缓存将尝试回收最近没有使用或总体上很少使用的缓存项。

```java
	final static Cache<String, User> cache = CacheBuilder.newBuilder()
			.maximumSize(1000)//缓冲最大容量
			.maximumWeight(10000)//最大权重值
			.weigher(new Weigher<String, User>() {//权重计算方式

				@Override
				public int weigh(String key, User user) {
					return user.getAge();
				}
			})
			.build();
```
### 2.2 定时回收（Timed Eviction）

CacheBuilder提供两种定时回收的方法：

- expireAfterAccess(long, TimeUnit)：缓存项在给定时间内没有被读/写访问，则回收。请注意这种缓存的回收顺序和基于大小回收一样。
- expireAfterWrite(long,TimeUnit)：缓存项在给定时间内没有被写访问（创建或覆盖），则回收。

> expireAfterWrite如果认为缓存数据总是在固定时候后变得陈旧不可用，这种回收方式是可取的。

### 2.3 基于引用的回收（Reference-based Eviction）

通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以把缓存设置为允许垃圾回收：



































