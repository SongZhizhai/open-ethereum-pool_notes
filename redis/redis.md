# open-ethereum-pool以太坊矿池-storage模块(redis存储)

## RedisClient定义

```go
type RedisClient struct {
        client *redis.Client
        prefix string
}
```

## Redis Key

* eth:nodes
	* eth:nodes:main:name
	* eth:nodes:main:height
	* eth:nodes:main:difficulty
	* eth:nodes:main:lastBeat
* eth:pow
	* ZADD eth:pow height params
* eth:stats
	* HINCRBY eth:stats roundShares diff
	* HDEL eth:stats roundShares
	* HGETALL eth:stats
	* HSET eth:stats lastBlockFound ts
* eth:shares
	* eth:shares:roundCurrent
		* HINCRBY eth:shares:roundCurrent login diff
		* RENAME eth:shares:roundCurrent eth:shares:round&height:nonce
		* HGETALL eth:shares:round&height:nonce
* eth:hashrate
	* eth:hashrate
		* ZADD eth:hashrate ts diff:login:id:ms
		* ZREMRANGEBYSCORE eth:hashrate -inf (now-window
		* ZRANGE eth:hashrate 0 -1 WITHSCORES
	* eth:hashrate:login
		* ZADD eth:hashrate:login ts diff:login:id:ms
		* EXPIRE eth:hashrate:login expire
* eth:miners
	* eth:miners:login
		* HSET eth:miners:login lastShare ts
		* HINCRBY eth:miners:login blocksFound 1
* eth:finders
	* ZINCRBY eth:finders 1 login
* eth:blocks
	* eth:blocks:candidates
		* ZADD eth:blocks:candidates height nonce:powHash:mixDigest:MakeTimestamp:h.diff:totalShares
		* ZREVRANGE eth:blocks:candidates 0 -1 WITHSCORES
		* ZCARD eth:blocks:candidates
	* eth:blocks:immature
		* ZREVRANGE eth:blocks:immature 0 -1 WITHSCORES
		* ZCARD eth:blocks:immature
	* eth:blocks:matured
		* ZREVRANGE eth:blocks:matured 0 49 WITHSCORES
		* ZCARD eth:blocks:matured
* eth:payments
	* eth:payments:all
		* ZREVRANGE eth:payments:all 0 49 WITHSCORES
		* ZCARD eth:payments:all

## Redis数据调取

```
redis-cli -h 127.0.0.1 -p 6379

127.0.0.1:6379> hgetall eth:nodes
1) "main:name"
2) "main"
3) "main:height"
4) "2753495"
5) "main:difficulty"
6) "551373957"
7) "main:lastBeat"
8) "1519977829"
//补充main:lastBeat为写入时间
```

## 参考文档

* [MULTI](http://redisdoc.com/transaction/multi.html)
* [Redis Hset 命令](http://www.runoob.com/redis/hashes-hset.html)
* [gopkg.in/redis.v3](https://godoc.org/gopkg.in/redis.v3)