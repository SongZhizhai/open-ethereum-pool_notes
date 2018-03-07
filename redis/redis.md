# open-ethereum-pool以太坊矿池-storage模块(redis存储)

## Redis基础

### Set

```
Set：String 类型的无序集合，集合成员是唯一的
SMEMBERS key 返回集合中的所有成员
```

## sorted set

```
sorted set：有序集合和集合一样也是string类型元素的集合,且不允许重复的成员
不同的是每个元素都会关联一个double类型的分数
ZRANGE key start stop [WITHSCORES] 通过索引区间返回有序集合成指定区间内的成员
ZREVRANGE key start stop [WITHSCORES] 返回有序集中指定区间内的成员，通过索引，分数从高到底
ZCARD key 获取有序集合的成员数

ZREMRANGEBYSCORE key min max 移除有序集合中给定的分数区间的所有成员
ZADD key score1 member1 [score2 member2] 向有序集合添加一个或多个成员，或者更新已存在成员的分数
ZINCRBY key increment member  有序集合中对指定成员的分数加上增量 increment
ZREM key member [member ...] 移除有序集合中的一个或多个成员
```

### Hash

```
Hash：是一个string类型的field和value的映射表，hash特别适合用于存储对象
HGETALL key 获取在哈希表中指定 key 的所有字段和值
HGET key field 获取存储在哈希表中指定字段的值

HSET key field value 将哈希表 key 中的字段 field 的值设为 value
HINCRBY key field increment 为哈希表 key 中的指定字段的整数值加上增量 increment 
HDEL key field1 [field2] 删除一个或多个哈希表字段
```

### 键(key)

```
SCAN cursor [MATCH pattern] [COUNT count] 用于迭代当前数据库中的数据库键

RENAME key newkey 修改 key 的名称
EXPIRE key seconds 为给定 key 设置过期时间
DEL key 该命令用于在 key 存在时删除 key
EXISTS key 检查给定 key 是否存在
```

### 字符串(String)

```
GET key 获取指定 key 的值

SETNX key value 只有在 key 不存在时设置 key 的值
```

### Redis 事务

```
WATCH key [key ...] 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断
MULTI 标记一个事务块的开始
EXEC 取消 WATCH 命令对所有 key 的监视
```

## Redis数据结构

### 黑名单eth:blacklist

```
类型：Set
SMEMBERS eth:blacklist
```

### 白名单eth:whitelist

```
类型：Set
SMEMBERS eth:whitelist
```

### 节点状态eth:nodes

```
类型：Hash
HGETALL eth:nodes

//如下为更新操作，慎重使用
HSET eth:nodes <name>:name <name>
HSET eth:nodes <name>:height height
HSET eth:nodes <name>:difficulty diff
HSET eth:nodes <name>:lastBeat now
```

### 统计eth:stats

```
类型：Hash
HGETALL eth:stats

//如下为更新操作，慎重使用
HSET eth:stats lastBlockFound ts
HINCRBY eth:stats roundShares diff
HDEL eth:stats roundShares
```

### 矿机eth:miners:login

```
类型：Hash
SCAN eth:miners:* 100
HGETALL eth:miners:login
HGET eth:miners:login balance

//如下为更新操作，慎重使用
HINCRBY eth:miners:login blocksFound 1
HSET eth:miners:login lastShare ts

//如下为更新操作，慎重使用
HINCRBY eth:miners:login balance -amount
HINCRBY eth:miners:login pending amount
//如下两行为回滚
HINCRBY eth:miners:login balance amount
HINCRBY eth:miners:login pending -amount

HINCRBY eth:miners:login paid amount
HINCRBY eth:miners:login immature amount
HINCRBY eth:miners:login immature -amount

EXISTS eth:miners:login
```

### 财务eth:finances

```
类型：Hash
HGETALL eth:finances

//如下为更新操作，慎重使用
HINCRBY eth:finances balance -amount
HINCRBY eth:finances pending amount
//如下两行为回滚
HINCRBY eth:finances balance amount
HINCRBY eth:finances pending -amount

HINCRBY eth:finances paid amount
HINCRBY eth:finances immature total

HSET eth:finances lastCreditHeight blockHeight
HSET eth:finances lastCreditHash blockHash
HSET eth:finances totalMined blockRewardInShannon
```

### Shares eth:shares

```
类型：Hash
HGETALL eth:shares:roundCurrent
HGET eth:shares:roundCurrent login
HGETALL eth:shares:round<height>:nonce

//如下为更新操作，慎重使用
HINCRBY eth:shares:roundCurrent login diff
RENAME eth:shares:roundCurrent eth:shares:round<height>:nonce
```

### 信用事务 eth:credits:immature:Height:Hash

```
类型：Hash
HGETALL eth:credits:immature:Height:Hash

//如下为更新操作，慎重使用
SETNX eth:credits:immature:Height:Hash login amount
WATCH eth:credits:immature:Height:Hash
```

### 提交Share eth:pow

```
类型：sorted set
ZRANGEBYSCORE eth:pow 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZREMRANGEBYSCORE eth:pow -inf (height-8
ZADD eth:pow height nonce:powHash:mixDigest
```

### 爆块者eth:finders

```
类型：sorted set
ZRANGEBYSCORE eth:finders 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZINCRBY eth:finders 1 login
```

### 候选块eth:blocks:candidates

```
类型：sorted set
ZRANGEBYSCORE eth:blocks:candidates 0 -1 WITHSCORES
ZRANGEBYSCORE eth:blocks:candidates 0 maxHeight WITHSCORES
ZCARD eth:blocks:candidates

//如下为更新操作，慎重使用
ZADD eth:blocks:candidates height hashHex:ts:roundDiff:totalShares
ZREM eth:blocks:candidates candidateKey
```

### 未成年块 eth:blocks:immature

```
类型：sorted set
ZRANGEBYSCORE eth:blocks:candidates 0 -1 WITHSCORES
ZREVRANGE eth:blocks:immature 0 -1 WITHSCORES
ZCARD eth:blocks:immature

ZRANGEBYSCORE eth:blocks:candidates 0 maxHeight WITHSCORES

//如下为更新操作，慎重使用
ZADD eth:blocks:immature Height key()
ZREM eth:blocks:immature immatureKey
```

### 成熟块 eth:blocks:matured

```
类型：sorted set
ZRANGEBYSCORE eth:blocks:matured 0 maxBlocks-1 WITHSCORES
ZREVRANGE eth:blocks:matured 0 max-1 WITHSCORES
ZCARD eth:blocks:matured

//如下为更新操作，慎重使用
ZADD eth:blocks:matured Height key()
```

### 算力统计 eth:hashrate

```
类型：sorted set
ZRANGEBYSCORE eth:hashrate 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZADD eth:hashrate ts diff:login:id:ms
ZREMRANGEBYSCORE eth:hashrate -inf (now-window 

SCAN eth:hashrate:* 100
ZRANGEBYSCORE eth:hashrate:login 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZADD eth:hashrate:login ts diff:id:ms
EXPIRE eth:hashrate:login expire
ZREMRANGEBYSCORE eth:hashrate:login -inf (now-largeWindow 
```

### 未支付 eth:payments:pending

```
类型：sorted set
ZREVRANGE eth:payments:pending 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZADD eth:payments:pending ts login:amount
//如下一行为回滚
ZREM eth:payments:pending login:amount
```

### 所有支付 eth:payments:all

```
类型：sorted set
ZRANGEBYSCORE eth:payments:all 0 maxPayments-1 WITHSCORES
ZCARD eth:payments:all

//如下为更新操作，慎重使用
ZADD eth:payments:all ts txHash:login:amount
```

### 单个支付 eth:payments:login

```
类型：sorted set
ZREVRANGE eth:payments:login 0 maxPayments-1 WITHSCORES
ZCARD eth:payments:login

//如下为更新操作，慎重使用
ZADD eth:payments:all ts txHash:login:amount
```

### 信用 eth:credits:all

```
类型：sorted set
ZREVRANGE eth:credits:all 0 -1 WITHSCORES

//如下为更新操作，慎重使用
ZADD eth:credits:all Height blockHash:ts:blockReward
SETNX eth:credits:Height:blockHash login amount
```

### 支付锁eth:payments:lock

```
类型：String
GET eth:payments:lock 

//如下为更新操作，慎重使用
SETNX eth:payments:lock login:amount 0
DEL eth:payments:lock
```

## 参考文档

* [Redis 集合(Set)](http://www.runoob.com/redis/redis-sets.html)
* [Redis 有序集合(sorted set)](http://www.runoob.com/redis/redis-sorted-sets.html)
* [
* [MULTI](http://redisdoc.com/transaction/multi.html)
* [Redis Hset 命令](http://www.runoob.com/redis/hashes-hset.html)
* [gopkg.in/redis.v3](https://godoc.org/gopkg.in/redis.v3)