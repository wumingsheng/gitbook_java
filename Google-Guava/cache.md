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
  1. CacheLoader
  2. Callable

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

## 1.2 Callable 

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





