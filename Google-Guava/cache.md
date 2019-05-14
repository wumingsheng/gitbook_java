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





