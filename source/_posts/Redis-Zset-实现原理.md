---
title: Redis Zset 实现原理
date: 2019-06-28 09:30:23
tags: Redis
---

# Redis TTL

Redis 只在key一级上有TTL (Time to live) 机制，[https://redis.io/commands/expire](https://redis.io/commands/expire "Expire Command")，有的时候我们需要把item按某种顺序排序，如排行榜或移动时间窗口，这个时候使用Sorted Set(Zset)就非常有用了。

# Redis Zset 按时间过期机制

_1. zadd 添加item时设置score为当前unixtime
__1.1. 如果只以第一次插入时间戳做删除则指定nx=true, [https://redis.io/commands/zadd](https://redis.io/commands/zadd)
__1.2. 如果需要更新已经存在item时间戳为当前时间TTL则指定默认选项，score设置为当前unittime
__1.3. 如果每次需要延长固定时间，检查item存在时使用ZINCRBY命令
_2. 定时做清理任务，每次计算删除某个时间点之前的所有item  zremrangebyscore


# Redis Zset 命令时间复杂度

Command Complexities of Zset and Set

|                      	| Zset        	| Set  	|
|----------------------	|-------------	|------	|
| Add                  	| O(log(N))   	| O(1) 	|
| Delete (1 items)     	| O(log(N))   	| O(1) 	|
| Iterate items by key 	| O(log(N)+M) 	| O(N) 	|
| Scan by score        	| O(log(N)+M) 	| N/A  	|

可以看出zset单个元素操作一般为Log(N)其中N为当前集合元素个数。

# Redis Zset 底层结构及算法实现
与set无顺序存储不同，Zset按score顺序进行存储，这也是为什么基本操作都是O(log(N))复杂度。

Redis使用两种结构存储zset，在数据个数较少时使用ziplist，数量超出阈值时使用skiplist，阈值通过zset-max-ziplist-entries and zset-max-ziplist-value设置。

## ziplist
ziplist使用连续空间存储双向链表，相比基于堆空间指针的链表前后向移动速度更快。

![](https://redislabs.com/wp-content/images/academy/redis-in-action/RIA_fig9-01.svg)

reids list也是使用ziplist存储。

## skiplist
skiplist 跳跃列表:

    /* ZSETs use a specialized version of Skiplists */
	typedef struct zskiplistNode {
	    sds ele;
	    double score;
	    struct zskiplistNode *backward;
	    struct zskiplistLevel {
	        struct zskiplistNode *forward;
	        unsigned long span;
	    } level[];
	} zskiplistNode;
	
	typedef struct zskiplist {
	    struct zskiplistNode *header, *tail;
	    unsigned long length;
	    int level;
	} zskiplist;
	
	typedef struct zset {
	    dict *dict;
	    zskiplist *zsl;
	} zset;

跳跃列表保存一个有序排列的链表，通过采用多层存储且保持每一层链表是其上一层链表的自己，从最稀疏的层开始搜索，从而达到比链表O(N)更优的查找和插入性能O(log(N))。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Skip_list_add_element-en.gif/600px-Skip_list_add_element-en.gif)

具体说明参见[wiki](https://en.wikipedia.org/wiki/Skip_list)