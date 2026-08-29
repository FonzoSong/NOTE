### 关于将 memcached 的数据摘要算法（hash）调整为md5
version: memcached 1.5.6

有关的help文档：
```shell
-o , --extended
...
- hash_algorithm:      the hash table algorithm
                          default is murmur3 hash. options: jenkins, murmur3
...                          
```
那么指定hash算法为md5是不可以的；只能指定为 jenkins, murmur3；本位使用jenkins配置;

### 修改配置文件
PATH: `/etc/sysconfig/memcached`
```conf
PORT="11211"
USER="memcached"
MAXCONN="2048"
CACHESIZE="256"
OPTIONS="127.0.0.1,::1,controller"
HASH="hash_algorithm=jenkins"
```
### 修改service文件
```conf
[Service]
EnvironmentFile=/etc/sysconfig/memcached
ExecStart=/usr/bin/memcached  -p ${PORT} -u ${USER} -m ${CACHESIZE} -c ${MAXCONN} -o ${HASH} -l ${OPTIONS} 
```
这样就可以使用 `systemctl` 来进行控制了;


#### PS：
##### Jenkins 哈希算法
###### 概述
Jenkins 哈希算法是由 Bob Jenkins 设计的一系列非加密哈希函数。这些算法主要用于数据结构和数据库中的散列表，以及一致性哈希等场景。

###### 主要特点
高性能：Jenkins 哈希算法在大多数情况下具有很高的性能，尤其是在处理较长的输入时。

均匀分布：Jenkins 哈希算法能够产生均匀分布的哈希值，这对于减少哈希冲突非常重要。

简单实现：Jenkins 哈希算法的实现相对简单，易于理解和实现。

多种变体：Jenkins 设计了多种哈希函数，包括 lookup3、spookyhash 等，适用于不同的应用场景。
###### 常见变体
lookup3：适用于短字符串和长字符串，具有良好的性能和均匀分布。

spookyhash：适用于长字符串，具有更高的性能和更好的均匀分布。
##### Murmur3 哈希算法
###### 概述
Murmur3 是由 Austin Appleby 设计的一种非加密哈希函数。它是 Murmur 哈希算法系列的第三个版本，广泛应用于各种数据结构和数据库中。

###### 主要特点
高性能：Murmur3 在大多数情况下具有极高的性能，尤其是在处理较长的输入时。

均匀分布：Murmur3 能够产生均匀分布的哈希值，减少了哈希冲突的概率。

低碰撞率：Murmur3 的设计使得不同输入产生相同哈希值的概率非常低。

多种变体：Murmur3 有两种主要变体：32位和128位哈希值。
###### 常见变体
Murmur3 32-bit：适用于需要较小哈希值的场景，计算速度快。
Murmur3 128-bit：适用于需要较大哈希值的场景，具有更高的唯一性和均匀分布。
#### 对比
##### 性能
Jenkins 和 Murmur3 都具有很高的性能，但在某些情况下，Murmur3 可能会稍微快一些，尤其是在处理较长的输入时。
##### 分布均匀性
Jenkins 和 Murmur3 都能产生均匀分布的哈希值，但在某些测试中，Murmur3 的分布可能更均匀。
##### 实现复杂度
Jenkins 的实现相对简单，易于理解和实现。
Murmur3 的实现稍微复杂一些，但仍然易于理解。
##### 使用场景
Jenkins：适用于需要高性能和均匀分布的场景，特别是在处理短字符串时。

Murmur3：适用于需要高性能和低碰撞率的场景，特别是在处理长字符串时。