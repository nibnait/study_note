---
title: Redis 都有哪些数据结构？
tags: 
  - Redis
categories: 读书笔记
top: 0
copyright: ture
date: 2019-07-06 19:57:45
mathjax: true
---
字符串String，字段Hash，列表List，集合Set，有序集合ZSet。

位图、[HyperLogLog](../../learning/HyperLogLog)、[GeoHash](../../learning/GeoHash)、PubSub、Stream

Redis Module：RedisBloom、RedisSearch、Redis-Cell

<!--more-->

# 位图

用于统计用户进1年的签到情况。它的本质还是String，只是Redis天然支持位操作。（直接将byte数组看成「位数组」即可）

> setbit key offset value
>
> getbit key offset
>
> bitcount key [start] [end]	#统计两个offset之间1的个数
>
> bittop op destkey [key…]	#对两个位图进行and、or、not、xor操作，将结果保存在destkey中
>
> bitfield key get/set/incrby



# HyperLogLog

用于统计页面访问UV、直播在线人数、微博点赞数、微博词条搜索次数，等这些对统计精度要求不是100%，且不需要返回统计集合内所有的元素的场景。
原理：$$N = 2^{k_{max}}$$，利用每个桶内的$$k_{max}$$的调和平均值估计集合的整体基数。

> pfadd key element
>
> pfcount key
>
> pfmerge [key…]

详情见[HyperLogLog Counting算法](../../learning/HyperLogLog)

# GeoHash

Redis 在 3.2 版本以后增加了地理位置 GEO 模块，意味着我们可以使用 Redis 来实现摩拜单车「附近的 Mobike」、饿了么「附近的餐馆」这样的功能了。  
业界比较通用的地理位置距离排序算法是 [GeoHash 算法](../../learning/GeoHash)，Redis 也使用 GeoHash 算法。它将二维的经纬度数据映射成一维的整数，这样所有的元素都将在挂载到一条线上，距离靠近的二维坐标映射到一维后的点之间距离也会很接近。  
在使用 Redis 进行 Geo 查询时，它的内部结构实际上只是一个 zset(skiplist)。通过 zset 的 score 排序就可以得到坐标附近的其它元素，通过将 score 还原成坐标值就可以得到元素的原始坐标。

>geoadd key (longitude) (latitude) (address)
>
>geodist key (addressA) (addressB) (km)
>
>geopos key address	#获取元素的经纬度信息
>
>geohash key address	#获取元素的hash值
>
>georadiusbymember key address 20 km [withcoord] [withdist] [withhash] count 3 asc # 查询附近20km内最近的3个元素，正序并显示经纬度、距离、hash信息

# PubSub（消息多播）

可以多个客户端订阅同一个消息主题，实现消息多播。但是此模式的缺点就是一旦Redis停机重启，PubSub 的消息是不会持久化的，毕竟 Redis 宕机就相当于一个消费者都没有，所有的消息直接被丢弃。

Redis5.0 新增的 Stream 数据结构，这个功能给 Redis 带来了持久化消息队列，从此 PubSub 可以消失了

# Stream

Stream 的消费模型借鉴了 Kafka 的消费分组的概念，它弥补了 Redis Pub/Sub 不能持久化消息的缺陷。但是它又不同于 kafka，Kafka 的消息可以分 partition，而 Stream 不行。 如果非要分 parition 的话，得在客户端做，提供不同的 Stream 名称，对消息进行 hash 取模来选择往哪个 Stream 里塞。

# RedisBloom（布隆过滤器）

它说有，不一定有。它说没有，就一定没有。

因为布隆过滤器的实现原理是利用一个长度为l的位数组，没bf.add一个元素进来，会经过k个hash函数，得到k个不同的位置，然后将这k个位置的数字置位1。当调用bf.exists时，直接去这个key得到的k个hash位置判断是否都为1，有一个位置为0，这判为不存在。但是即使都为0，这个key也不一定存在，因为这k个hash位也有可能是由其他若干个key置位1的。

> bf.add key value	# bf.madd key [value...]
>
> bf.exists key value	# bf.mexists key [value...]

## 使用场景

**垃圾邮件的过滤**、今日头条**推荐文章去重**、判断一个商户是不是我这个平台的（防止缓存穿透）、**爬虫**系统**url去重**（当爬取网页url的数量达到几千万几亿的级别时）等等这种数据量很大，判断key是否存在（去重、过滤），且允许一定几率误判的业务场景。

## 调优

Redis 官方提供的布隆过滤器到了 Redis 4.0 提供了插件功能之后才正式登场。在执行bf.add(key)之前会使用bf.reserve执行显示创建BoolmFIlter，两个个关键参数：
 - f（error_rate(0.01)）数字越小，占用空间越大。
 - n（initial_size(100)）当key的实际数量超出这个数值时，误判率会上升。

这两个会直接影响布隆过滤器的数组长度(l)、和hash函数的最佳数量(k)
简单的计算公式如下：

 - k ≈ 0.7 * $$\frac{l}{n}$$
 - f = $$0.6185^\frac{l}{n}$$

公式的证明：[数学学渣慎点](https://www.cnblogs.com/allensun/archive/2011/02/16/1956532.html)  
线上计算空间占用：[https://krisives.github.io/bloom-calculator/](https://krisives.github.io/bloom-calculator/)

误判为什么会上升？

# RedisSearch（全文搜索引擎）

# Redis-Cell