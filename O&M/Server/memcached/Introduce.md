# memcached与redis区别：

1. Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例片、视频等等
2. Redis 不仅仅支持简单的k/v 类型的数据，同时还提供list，set，zset，hash等数据结构的存储。而memcache 只支持简单数据类型，需要客户端自己处理复杂对象。
3. redis的速度比memcached快很多
4. redis可以持久化其数据
5. Redis支持数据的备份，即master-slave模式的数据备份。
6. 使用底层模型不同，它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
7. value大小：redis最大可以达到1GB，而memcache只有1MB

#  what is MemCache
MemCache是一个自由、源码开放、高性能、分布式的分布式内存对象缓存系统，用于动态Web应用以减轻数据库的负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高了网站访问的速度。MemCaChe是一个存储键值对的HashMap，在内存中对任意的数据（比如字符串、对象等）所使用的key-value存储，数据可以来自数据库调用、API调用，或者页面渲染的结果。MemCache设计理念就是小而强大，它简单的设计促进了快速部署、易于开发并解决面对大规模的数据缓存的许多难题，而所开放的API使得MemCache能用于Java、C/C++/C#、Perl、Python、PHP、Ruby等大部分流行的程序语言。

MemCache的官方网站为http://memcached.org/
