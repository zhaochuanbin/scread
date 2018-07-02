---
layout : post
title : "redis常用命令"
category : redis
tagline: ""
date : 2016-06-14
tags : [nosql,redis]
---

### Connection命令

    echo message - 回显输入的字符串
    ping - ping服务器
    quit - 关闭连接,退出
	select index - 选择新数据库
	CONFIG GET loglevel - 查询日志等级
	CONFIG GET * 查询所有配置设置

### keys命令
	exits key 测试指定key是否存在，返回1表示存在，0不存在
	del key1 key2 ....keyN  删除给定key,返回删除key的数目，0表示给定key都不存在
	type key 返回给定key的value类型。返回 none 表示不存在
	keys pattern 返回匹配指定模式的所有key（支持*，？，[abc ]的方式）
	randomkey 返回从当前数据库中随机选择的一个key,如果当前数据库是空的，返回空串
	rename oldkey newkey 原子的重命名一个key,如果newkey存在，将会被覆盖，返回1表示成功，0失败。失败可能是oldkey不存在或者和newkey相同
	renamenx oldkey newkey 同上，但是如果newkey存在返回失败
	dbsize 返回当前数据库的key数量
	expire key seconds 为key指定过期时间，单位是秒。返回1成功，0表示key已经设置过过期时间或者不存在
	ttl key 返回设置了过期时间的key的剩余过期秒数， -1表示key不存在或者没有设置过过期时间
	select db-index 通过索引选择数据库，默认连接的数据库所有是0,默认数据库数是16个。返回1表示成功，0失败
	move key db-index  将key从当前数据库移动到指定数据库。返回1成功。0 如果key不存在，或者已经在指定数据库中
	flushdb 删除当前数据库中所有key,此方法不会失败。慎用
	flushall 删除所有数据库中的所有key，此方法不会失败。更加慎用

### 字符串命令
- string类型是二进制安全的，最大长度为 512MB，
- 可以包含任何数据，包括jpg图片或者序列化的对象


	set key value 设置key对应的值为string类型的value,返回1表示成功，0失败
	setnx key value 同上，如果key已经存在，返回0 。nx 是not exist的意思
	get key 获取key对应的string值,如果key不存在返回nil
	getset key value  设置key的值，并返回key的旧值。如果key不存在返回nil
	mget key1 key2 ... keyN 一次获取多个key的值，如果对应key不存在，则对应返回nil。下面是个实验, nonexisting不存在，对应返回nil
	mset key1 value1 ... keyN valueN 一次设置多个key的值，成功返回1表示所有的值都设置了，失败返回0表示没有任何值被设置
	msetnx key1 value1 ... keyN valueN 同上，但是不会覆盖已经存在的key
	incr key 对key的值做加加操作,并返回新的值。注意incr一个不是int的value会返回错误，incr一个不存在的key，则设置key为1
	decr key 同上，但是做的是减减操作，decr一个不存在key，则设置key为-1
	incrby key integer 同incr，加指定值 ，key不存在时候会设置key，并认为原来的value是 0
	incrbyfloat key float 同incr，加指定值 ，key不存在时候会设置key，并认为原来的value是 0
	decrby key integer 同decr，减指定值。decrby完全是为了可读性，我们完全可以通过incrby一个负值来实现同样效果，反之一样。
	append key value  给指定key的字符串值追加value,返回新字符串值的长度。
	substr key start end 返回截取过的key的字符串值,注意并不修改key的值。下标是从0开始的


### 散列/哈希
-是一个string类型的field和value的映射表，存储多达2^32 - 1个健,一般存对象
- 将一个对象存储在hash类型中会占用更少的内存，并且可以更方便的存取整个对象。


	hset key field value 设置hash field为指定值，如果key不存在，则先创建
	hget key field  获取指定的hash field
	hmget key filed1....fieldN 获取全部指定的hash filed
	hmset key filed1 value1 ... filedN valueN 同时设置hash的多个field
	hincrby key field integer 将指定的hash filed 加上给定值
	hexists key field 测试指定field是否存在
	hdel key field 删除指定的hash field
	hlen key 返回指定hash的field数量
	hkeys key 返回hash的所有field
	hvals key 返回hash的所有value
	hgetall key 返回hash的所有filed和value

列表:
- 是一个每个子元素都是string类型的双向链表,按插入顺序排序,头部或尾部添加元素
- 这使得list既可以用作栈，也可以用作队列.最大长度为2^32 - 1
- list则可以阻塞，当然可以加超时时间，超时后也会返回nil，主要是为了避免轮询


	lpush key string 在key对应list的头部添加字符串元素，返回1表示成功，0表示key存在且不是list类型
	rpush key string 同上，在尾部添加
	llen key 返回key对应list的长度，key不存在返回0,如果key对应类型不是list返回错误
	lrange key start end 返回指定区间内的元素，下标从0开始，负值表示从后面计算，-1表示倒数第一个元素 ，key不存在返回空列表
	ltrim key start end  截取list，保留指定区间内元素，成功返回1，key不存在返回错误
	lset key index value 设置list中指定下标的元素值，成功返回1，key或者下标不存在返回错误
	lrem key count value 从key对应list中删除count个和value相同的元素。count为0时候删除全部
	lpop key 从list的头部删除元素，并返回删除元素。如果key对应list不存在或者是空返回nil，如果key对应值不是list返回错误
	rpop 同上，但是从尾部删除
	blpop key1...keyN timeout 从左到右扫描返回对第一个非空list进行lpop操作并返回，比如blpop list1 list2 list3 0 ,如果list不存在，list2,list3都是非空则对list2做lpop并返回从list2中删除的元素。如果所有的list都是空或不存在，则会阻塞timeout秒，timeout为0表示一直阻塞。当阻塞时，如果有client对key1...keyN中的任意key进行push操作，则第一在这个key上被阻塞的client会立即返回。如果超时发生，则返回nil。
	brpop 同blpop，一个是从头部删除一个是从尾部删除
	rpoplpush srckey destkey 从srckey对应list的尾部移除元素并添加到destkey对应list的头部,最后返回被移除的元素值，整个操作是原子的.如果srckey是空或者不存在返回nil

无序集合:
- set是string类型的无序集合,
- 添加，删除和测试成员存在的时间O(1)复杂性,由于集合的唯一属性，所以它只算添加一次
- 是通过hash table实现的，hash table会随着添加或者删除自动的调整大小
- 集合的取并集(union)，交集(intersection)，差集(difference)


	sadd key member 添加一个string元素到,key对应的set集合中，成功返回1,如果元素以及在集合中返回0,key对应的set不存在返回错误
	srem key member 从key对应set中移除给定元素，成功返回1，如果member在集合中不存在或者key不存在返回0，如果key对应的不是set类型的值返回错误
	spop key 删除并返回key对应set中随机的一个元素,如果set是空或者key不存在返回nil
	srandmember key 同spop，随机取set中的一个元素，但是不删除元素
	smove srckey dstkey member 从srckey对应set中移除member并添加到dstkey对应set中，整个操作是原子的。成功返回1,如果member在srckey中不存在返回0，如果key不是set类型返回错误
	scard key 返回set的元素个数，如果set是空或者key不存在返回0
	sismember key member 判断member是否在set中，存在返回1，0表示不存在或者key不存在
    sinter key1 key2...keyN 返回所有给定key的交集
	sinterstore dstkey key1...keyN 同sinter，但是会同时将交集存到dstkey下
	sunion key1 key2...keyN 返回所有给定key的并集
	sunionstore dstkey key1...keyN 同sunion，并同时保存并集到dstkey下
	sdiff key1 key2...keyN 返回所有给定key的差集
	sdiffstore dstkey key1...keyN 同sdiff，并同时保存差集到dstkey下
	smembers key 返回key对应set的所有元素，结果是无序的

### 有序集合
- 也是string类型元素的集合
- 每个元素都会关联一个double类型的score。sorted set的实现是skip list和hash table的混合体。当元素被添加到集合中时，一个元素到score的映射被添加到hash table中，另一个score到元素的映射被添加到skip list并按照score排序，所以就可以有序的获取集合中的元素。

	zadd key score member 添加元素到集合，元素在集合中存在则更新对应score
	zrem key member 删除指定元素，1表示成功，如果元素不存在返回0
	zincrby key incr member 增加对应member的score值，然后移动元素并保持skip list有序。返回更新后的score值
	zrank key member 返回指定元素在集合中的排名（下标，非score）,集合中元素是按score从小到大排序的
	zrevrank key member 同上,但是集合中元素是按score从大到小排序
	zrange key start end 类似lrange操作从集合中取指定区间的元素。返回的是有序结果
	zrevrange key start end 同上，返回结果是按score逆序的
	zrangebyscore key min max 返回集合中score在给定区间的元素
	zcount key min max 返回集合中score在给定区间的数量
	zcard key 返回集合中元素个数
	zscore key element  返回给定元素对应的score
	zremrangebyrank key min max 删除集合中排名在给定区间的元素
	zremrangebyscore key min max 删除集合中score在给定区间的元素

