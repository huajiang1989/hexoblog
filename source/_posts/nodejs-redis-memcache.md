---
title: Redis和Memcache的区别分析
category:  nodejs
tags: nodejs
---


简单区别：
1. Redis中，并不是所有的数据都一直存储在内存中的，这是和Memcached相比一个最大的区别。
2. redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。
3. Redis支持数据的备份，即master-slave模式的数据备份。
4. Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
<!--more-->
Redis在很多方面具备数据库的特征，或者说就是一个数据库系统，而Memcached只是简单的K/V缓存

下面是来自redis作者的说法（stackoverflow上面）。
```
You should not care too much about performances. Redis is faster per core with small values, but memcached is able to use multiple cores with a single executable and TCP port without help from the client. Also memcached is faster with big values in the order of 100k. Redis recently improved a lot about big values (unstable branch) but still memcached is faster in this use case. The point here is: nor one or the other will likely going to be your bottleneck for the query-per-second they can deliver.
You should care about memory usage. For simple key-value pairs memcached is more memory efficient. If you use Redis hashes, Redis is more memory efficient. Depends on the use case.
You should care about persistence and replication, two features only available in Redis. Even if your goal is to build a cache it helps that after an upgrade or a reboot your data are still there.
You should care about the kind of operations you need. In Redis there are a lot of complex operations, even just considering the caching use case, you often can do a lot more in a single operation, without requiring data to be processed client side (a lot of I/O is sometimes needed). This operations are often as fast as plain GET and SET. So if you don’t need just GEt/SET but more complex things Redis can help a lot (think at timeline caching).
```
有网友翻译如下[1]：
```
没有必要过多的关注性能。由于Redis只使用单核，而Memcached可以使用多核，所以在比较上，平均每一个核上Redis在存储小数据时比Memcached性能更高。而在100k以上的数据中，Memcached性能要高于Redis，虽然Redis最近也在存储大数据的性能上进行优化，但是比起Memcached，还是稍有逊色。说了这么多，结论是，无论你使用哪一个，每秒处理请求的次数都不会成为瓶颈。
你需要关注内存使用率。对于key-value这样简单的数据储存，memcache的内存使用率更高。如果采用hash结构，redis的内存使用率会更高。当然，这些都依赖于具体的应用场景。
你需要关注关注数据持久化和主从复制时，只有redis拥有这两个特性。如果你的目标是构建一个缓存在升级或者重启后之前的数据不会丢失的话，那也只能选择redis。
你应该关心你需要的操作。redis支持很多复杂的操作，甚至只考虑内存的使用情况，在一个单一操作里你常常可以做很多，而不需要将数据读取到客户端中（这样会需要很多的IO操作）。这些复杂的操作基本上和纯GET和POST操作一样快，所以你不只是需要GET/SET而是更多的操作时，redis会起很大的作用。
对于两者的选择还是要看具体的应用场景，如果需要缓存的数据只是key-value这样简单的结构时，我在项目里还是采用memcache，它也足够的稳定可靠。如果涉及到存储，排序等一系列复杂的操作时，毫无疑问选择redis。
```
其他：
```
1、 Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等。
2、Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。
3、虚拟内存–Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘
4、过期策略–memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10
5、分布式–设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从
6、存储数据安全–memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）
7、灾难恢复–memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复
8、Redis支持数据的备份，即master-slave模式的数据备份。
```
关于redis和memcache的不同，下面罗列了一些相关说法，供记录：

# redis和memecache的不同：
## 存储方式：
memecache 把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小
redis有部份存在硬盘上，这样能保证数据的持久性，支持数据的持久化（笔者注：有快照和AOF日志两种持久化方式，在实际应用的时候，要特别注意配置文件快照参数，要不就很有可能服务器频繁满载做dump）。
## 数据支持类型：
redis在数据支持上要比memecache多的多。
## 使用底层模型不同：
新版本的redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
## 运行环境不同：
redis目前官方只支持Linux 上去行，从而省去了对于其它系统的支持，这样的话可以更好的把精力用于本系统 环境上的优化，虽然后来微软有一个小组为其写了补丁。但是没有放到主干上

个人总结一下，有持久化需求或者对数据结构和处理有高级要求的应用，选择redis，其他简单的key/value存储，选择memcache。

# Memcached和Redis两种方案
## Memcached介绍
Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提供动态、数据库驱动网站的速度，现在已被LiveJournal、hatena、Facebook、Vox、LiveJournal等公司所使用。

## Memcached工作方式分析
许多Web应用都将数据保存到 RDBMS中，应用服务器从中读取数据并在浏览器中显示。 但随着数据量的增大、访问的集中，就会出现RDBMS的负担加重、数据库响应恶化、 网站显示延迟等重大影响。Memcached是高性能的分布式内存缓存服务器,通过缓存数据库查询结果，减少数据库访问次数，以提高动态Web等应用的速度、 提高可扩展性。下图展示了memcache与数据库端协同工作情况：
这里写图片描述
其中的过程是这样的：
1.检查用户请求的数据是缓存中是否有存在，如果有存在的话，只需要直接把请求的数据返回，无需查询数据库。
2.如果请求的数据在缓存中找不到，这时候再去查询数据库。返回请求数据的同时，把数据存储到缓存中一份。
3.保持缓存的“新鲜性”，每当数据发生变化的时候（比如，数据有被修改，或被删除的情况下），要同步的更新缓存信息，确保用户不会在缓存取到旧的数据。

Memcached作为高速运行的分布式缓存服务器，具有以下的特点：
```
1.协议简单
2.基于libevent的事件处理
3.内置内存存储方式
4.memcached不互相通信的分布式
```
如何实现分布式可拓展性？
Memcached的分布式不是在服务器端实现的，而是在客户端应用中实现的，即通过内置算法制定目标数据的节点，如下图所示：
这里写图片描述

## Redis 介绍
Redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、 list(链表)、set(集合)和zset(有序集合)。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步,当前 Redis的应用已经非常广泛，国内像新浪、淘宝，国外像 Flickr、Github等均在使用Redis的缓存服务。

## Redis 工作方式分析
Redis作为一个高性能的key-value数据库具有以下特征：
```
1.多样的数据模型
2.持久化
3.主从同步
```
Redis支持丰富的数据类型，最为常用的数据类型主要由五种：String、Hash、List、Set和Sorted Set。Redis通常将数据存储于内存中，或被配置为使用虚拟内存。Redis有一个很重要的特点就是它可以实现持久化数据，通过两种方式可以实现数据持久化：使用RDB快照的方式，将内存中的数据不断写入磁盘；或使用类似MySQL的AOF日志方式，记录每次更新的日志。前者性能较高，但是可能会引起一定程度的数据丢失；后者相反。 Redis支持将数据同步到多台从数据库上，这种特性对提高读取性能非常有益。

Redis如何实现分布式可拓展性？
2.8以前的版本：与Memcached一致，可以在客户端实现，也可以使用代理，twitter已开发出用于Redis和Memcached的代理Twemproxy 。
3.0 以后的版本：相较于Memcached只能采用客户端实现分布式存储，Redis则在服务器端构建分布式存储。Redis Cluster是一个实现了分布式且允许单点故障的Redis高级版本，它没有中心节点，各个节点地位一致，具有线性可伸缩的功能。如图给出Redis Cluster的分布式存储架构，其中节点与节点之间通过二进制协议进行通信，节点与客户端之间通过ascii协议进行通信。在数据的放置策略上，Redis Cluster将整个 key的数值域分成16384个哈希槽，每个节点上可以存储一个或多个哈希槽，也就是说当前Redis Cluster支持的最大节点数就是16384。
这里写图片描述

# 综合结论

应该说Memcached和Redis都能很好的满足解决我们的问题，它们性能都很高，总的来说，可以把Redis理解为是对Memcached的拓展，是更加重量级的实现，提供了更多更强大的功能。具体来说：

## 性能上：
性能上都很出色，具体到细节，由于Redis只使用单核，而Memcached可以使用多核，所以平均每一个核上Redis在存储小数据时比
Memcached性能更高。而在100k以上的数据中，Memcached性能要高于Redis，虽然Redis最近也在存储大数据的性能上进行优化，但是比起 Memcached，还是稍有逊色。

## 内存空间和数据量大小：
MemCached可以修改最大内存，采用LRU算法。Redis增加了VM的特性，突破了物理内存的限制。

## 操作便利上：
MemCached数据结构单一，仅用来缓存数据，而Redis支持更加丰富的数据类型，也可以在服务器端直接对数据进行丰富的操作,这样可以减少网络IO次数和数据体积。

## 可靠性上：
MemCached不支持数据持久化，断电或重启后数据消失，但其稳定性是有保证的。Redis支持数据持久化和数据恢复，允许单点故障，但是同时也会付出性能的代价。

## 应用场景：
Memcached：动态系统中减轻数据库负载，提升性能；做缓存，适合多读少写，大数据量的情况（如人人网大量查询用户信息、好友信息、文章信息等）。
Redis：适用于对读写效率要求都很高，数据处理业务复杂和对安全性要求较高的系统（如新浪微博的计数和微博发布部分系统，对数据安全性、读写要求都很高）。

# 需要慎重考虑的部分
1.Memcached单个key-value大小有限，一个value最大只支持1MB，而Redis最大支持512MB
2.Memcached只是个内存缓存，对可靠性无要求；而Redis更倾向于内存数据库，因此对对可靠性方面要求比较高
3.从本质上讲，Memcached只是一个单一key-value内存Cache；而Redis则是一个数据结构内存数据库，支持五种数据类型，因此Redis除单纯缓存作用外，还可以处理一些简单的逻辑运算，Redis不仅可以缓存，而且还可以作为数据库用
4.新版本（3.0）的Redis是指集群分布式，也就是说集群本身均衡客户端请求，各个节点可以交流，可拓展行、可维护性更强大。