---
title: Redis 过期key删除策略和内存淘汰策略
tags: 
  - Redis
categories: 读书笔记
top: 0
copyright: ture
date: 2019-07-13 09:28:53
mathjax: true
---
# 过期删除策略

当Redis中缓存的key过期了，Redis如何处理？
<!--more-->
## 惰性删除

在客户端访问 key 的时候，如果这个 key 设置了过期时间，redis 则对 key 的过期时间进行检查，如果过期了就立即删除。  
memcached只是用了惰性删除，而Redis同时使用了惰性删除与定期删除

## 定期扫描删除

redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，然后会**定时遍历**这个字典来删除到期的 key。  
这个定时任务由 src/expire.c 的 activeExpireCycle(ACTIVE_EXPIRE_CYCLE_SLOW) 函数来完成，它的大致步骤是：

1. 每次从过期字典中随机选取20个key。
2. 删除这20个key中过期的key。
3. 如果这20个key中过期key的比例超过了 25%，则重复步骤1。

```c
src/server.h 中定义的每次获取 key 的数量和比例。
#define ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP 20 /* Loopkups per loop. */
#define ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC 25 /* CPU max % for keys collection */
```

为了防止这个过期扫描出现循环过度，此算法加了一个timelimit变量，函数每次默认最多执行25ms  

```c
//默认情况下，timelimit=25000(纳秒)
timelimit = 1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100;	

if ((iteration & 0xf) == 0) { /* check once every 16 iterations. */
  //删除过期key，每迭代16次，检查一次timelimit
  elapsed = ustime()-start;	//ustime()精确到纳秒
  if (elapsed > timelimit) {
    timelimit_exit = 1;
    break;
  }
}
```

即使加了timelimit，默认每100ms执行一次activeExpireCycle。如果同一时间有大量key过期，在默认情况下 Redis 也会有至少 1 / 4 的时间被阻塞来执行这个activeExpireCycle()方法。为防止出现这种情况，可以在设置过期时间时 后面再个随机值：

```java
// 在目标过期时间上增加一天的随机时间
redis.expire_at(key, random.randint(24*3600) + expire_ts)
```

## 从库的过期策略

从库不会进行过期扫描，从库对过期的处理是被动的。主库在 key 到期时，会在 AOF 文件里增加一条 del 指令，同步到所有的从库，从库通过执行这条 del 指令来删除过期的 key。
因为指令同步是异步进行的，所以主库过期的 key 的 del 指令没有及时同步到从库的话，会出现主从数据的不一致，主库没有的数据在从库里还存在，比如在集群环境中分布式锁的算法漏洞就是因为这个同步延迟产生的。

## 单线程的 Redis，如何知道要运行定时任务？

这个定期扫描删除任务在由 server.c/serverCron() 方法调用，Redis 将 serverCron 方法称作“timer interrupt”（循环时间事件），它每秒的执行次数可以通过redis.conf `hz`选项（默认是10）来配置。  
serverCron() 除了定期扫描删除过期key，主要还执行：

- 更新服务器的各类统计信息，比如时间（server.lruclock）、内存占用、数据库占用情况等。
- 关闭和清理连接失效的客户端。
- 尝试进行 AOF 或 RDB 持久化操作。
- 如果服务器是主节点的话，对附属节点进行定期同步。
- 如果处于集群模式的话，对集群进行定期同步和连接测试。

# 内存淘汰策略

当Redis已用内存超过maxmemory限定时，怎么处理需要新写入且需要申请额外空间的数据？根据redis.conf `maxmemory-policy` 选项来做对应的清理：

- volatile-lru: 从设置了过期时间的key中，删除最久未使用的key
- allkeys-lru: 从所有key中，删除最近最少使用的key
- volatile-lfu: 从设置了过期时间的key中，删除使用频率最少的key
- allkeys-lfu: 从所有key中，删除使用频率最少的key
- volatile-random: 从设置了过期时间的key中，随机删除
- allkeys-random: 从所有key中，随机删除
- volatile-ttl: 从设置了过期时间的key中，删除马上要过期的key
- noeviction: （Redis的默认淘汰策略）不删除，在写操作时直接报错

当mem_used达到maxmemory时，所有的写请求都会调用 src/evict.c 的 freeMemoryIfNeeded(void) 方法，执行对应的策略尝试释放一些内存。

## 近似的 LRU 算法

Redis 中的LRU并不是真正的LRU（Least Recently Used），为了避免LRU算法中hash表的内存消耗，Redis 给每个key都增加了一个额外的小字段（lru）：一个24bit长度的，记录着最后一次被访问的时间戳”。

在执行 Redis LRU 算法时，从待删除的key空间中随机采样`maxmemory-samples`（默认是5）个key，淘汰最久未访问过的那个key（redis作者测试的结果是当maxmemory-samples = 10时，已经非常接近全量LRU的精准度了）。

```c
#define LRU_BITS 24
#define LRU_CLOCK_MAX ((1<<LRU_BITS)-1) /* Max value of obj->lru */
#define LRU_CLOCK_RESOLUTION 1000 /* LRU clock resolution in ms */
```

**注意：**LRU 时间戳的默认精度是 1000ms，24bit 可以表示的最长时间大约是194天。

- 如果一个 key 超过194天没有被访问，在计算空闲时间的时候，就会少算194天，造成一定的误差。
- 为了避免 fork 子进程后额外的内存消耗，当进行 bgsave 或 aof rewrite 时，lru访问时间是不更新的。

```c
/* 如果对象本身存的lru访问时间 比系统的当前lru clock还要大
 * 那么在计算当前对象的空闲时间的时候，就不能简单的lruclock - o->lru相减了，需要额外计算*/
unsigned long long estimateObjectIdleTime(robj *o) {
    unsigned long long lruclock = LRU_CLOCK();
    if (lruclock >= o->lru) {
        return (lruclock - o->lru) * LRU_CLOCK_RESOLUTION;
    } else {
        return (lruclock + (LRU_CLOCK_MAX - o->lru)) *
                    LRU_CLOCK_RESOLUTION;
    }
}
/* 如果当前 lru clock 的分辨率低于系统刷新频率，则使用系统的时间戳(server.lruclock)
 * 		否则调用系统的mstime()函数重新计算*/
unsigned int LRU_CLOCK(void) {
    unsigned int lruclock;
    if (1000/server.hz <= LRU_CLOCK_RESOLUTION) {
        atomicGet(server.lruclock,lruclock);
    } else {
        lruclock = getLRUClock();
    }
    return lruclock;
}
/* 当计算出的当前clock超出LRU_CLOCK_MAX时，就会再次从0开始数
 * server.lruclock也是通过此方法来获取的当前时间*/
unsigned int getLRUClock(void) {
    return (mstime()/LRU_CLOCK_RESOLUTION) & LRU_CLOCK_MAX;
}
```

在 Redis 3.0 之后，又增加了一个`eviction pool`的结构，eviction pool是一个数组，保存了之前随机选取的key及它们的idle时间，数组里面的key按idle时间升序排序，当内存满了需要淘汰数据时，会调用`dictGetSomeKeys`选取指定的数目的key，然后更新到eviction pool里面，如果新选取的key的idle时间比eviction pool里面idle时间最小的key还要小，那么就不会把它插入到eviction pool里面。这样就保证了对于某些历史选取的key的idle时间相对来说比较久，但是本次淘汰并没有被选中，因为出现了idle时间更久的key，那么在使用eviction pool的情况下，这种idle时间比较久的key淘汰概率增大了，因为它在eviction pool里面被保存下来，参与下轮淘汰，这个思路和`访问局部性原理`是契合的。

因此在只维护一个eviction pool带来的少量开销情况下，对算法效率的提升是比较明显的，效率的提升带来的是访问命中率的提升。

## Redis 的 LFU

LFU（Least Frequently Used）是在Redis4.0后出现的，LRU的最近最少使用实际上并不精确，考虑下面的情况，如果在|处删除，那么A距离的时间最久，但实际上A的使用频率要比B频繁，所以合理的淘汰策略应该是淘汰B。LFU就是为应对这种情况而生的。

```java
A~~A~~A~~A~~A~~A~~A~~A~~A~~A~~~|
B~~~~~B~~~~~B~~~~~B~~~~~~~~~~~B|
```

在 Redis 中每个对象都有24 bits空间来记录LRU/LFU信息：

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (高16位用来记录访问时间（单位为分钟），
                            *            低8位用来记录访问频率，简称counter). */
    int refcount;
    void *ptr;
} robj;
```

### counter计数器对数因子和衰减时间

由于 LFU 的基数器只有8bit，所能记录的最大数字为255。因此counter并不是线性增长的，而是用基于概率的对数计数器来实现。counter的增长速度有`lfu-log-factor`参数控制。但最终counter总会收敛于255，为了解决这个问题，redis还提供了`lfu-decay-time`衰减因子（单位为分钟），如果一个key长时间未被访问，counter就要减少，减少的幅度由衰减因子来控制。

```c
void updateLFU(robj *val) {
    unsigned long counter = LFUDecrAndReturn(val);
    counter = LFULogIncr(counter);
    val->lru = (LFUGetTimeInMinutes()<<8) | counter;
}
/* 高16位，当server.unixtime超出2^16时，再次从0开始计数 */
unsigned long LFUGetTimeInMinutes(void) {
    return (server.unixtime/60) & 65535;
}
/* redis的key的空闲时间(分钟) */
unsigned long LFUTimeElapsed(unsigned long ldt) {
    unsigned long now = LFUGetTimeInMinutes();
    if (now >= ldt) return now-ldt;
    return 65535-ldt+now;
}
/* 在key被访问时，在counter++之前，要根据lfu_decay_time对counter做适当的修减：
  num_periods = 空闲时间/lfu_decay_time > 0
  counter = (num_periods > counter) ? 0 : counter - num_periods;
 */
unsigned long LFUDecrAndReturn(robj *o) {
    unsigned long ldt = o->lru >> 8;
    unsigned long counter = o->lru & 255;
    unsigned long num_periods = server.lfu_decay_time ? LFUTimeElapsed(ldt) / server.lfu_decay_time : 0;
    if (num_periods)
        counter = (num_periods > counter) ? 0 : counter - num_periods;
    return counter;
}

/* 基于概率的对数计数器：
  1. 提取0到1之间的随机数R。
  2. 计算counter++的概率 P = 1/(old_value*lfu_log_factor+1).
  3. 如果R < P，则counter++
 */
#define LFU_INIT_VAL 5 //counter的初始值为5，以便给新对象一个机会去积累点击率
uint8_t LFULogIncr(uint8_t counter) {
    if (counter == 255) return 255;
    double r = (double)rand()/RAND_MAX;
    double baseval = counter - LFU_INIT_VAL;
    if (baseval < 0) baseval = 0;
    double p = 1.0/(baseval*server.lfu_log_factor+1);
    if (r < p) counter++;
    return counter;
}
```

- 计数器对数因子`lfu-log-factor`默认是10，有redis.conf中的这张表可知，默认情况下，当访问次数达到100万时，counter才会增长到255。
- 计数器衰减时间`lfu-decay-time`是一个被衰减的key计数器所必须经过的时间（单位为分钟），默认是1。如果lfu-decay-time = 0，则代表每次在更新counter时，它都要被衰减。

```python
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
```

## TTL

跟近似的LRU一样，并不是去所有key中寿命最短的那个，而是每次随机取`maxmemory-samples`个key，删除这里面最即将过期的key。同样在Redis 3.0 之后由于引入了`eviction pool`数组，使内存淘汰key的策略更公平。

参考资料：

 - [Redis作为LRU Cache的实现](https://yq.aliyun.com/articles/63034)
 - [Redis近似LRU算法优化](https://yq.aliyun.com/articles/64435)
 - [Redis · 引擎特性 · 基于 LFU 的热点 key 发现机制](https://yq.aliyun.com/articles/643758)