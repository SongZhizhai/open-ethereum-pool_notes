# open-ethereum-pool以太坊矿池-redis存储

## WriteNodeState

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

每3s写入redis（写入间隔由proxy.stateUpdateInterval指定）

```
func NewProxy(cfg *Config, backend *storage.RedisClient) *ProxyServer {
	//部分代码略
	stateUpdateIntv := util.MustParseDuration(cfg.Proxy.StateUpdateInterval)
	stateUpdateTimer := time.NewTimer(stateUpdateIntv)
	//部分代码略
	go func() {
		for {
			select {
			case <-stateUpdateTimer.C:
				t := proxy.currentBlockTemplate()
				if t != nil {
					err := backend.WriteNodeState(cfg.Name, t.Height, t.Difficulty)
					if err != nil {
						log.Printf("Failed to write node state to backend: %v", err)
						proxy.markSick()
					} else {
						proxy.markOk()
					}
				}
				stateUpdateTimer.Reset(stateUpdateIntv)
			}
		}
	}()
	//部分代码略
}
//代码位置proxy/proxy.go
```

## 

## 参考文档

* [MULTI](http://redisdoc.com/transaction/multi.html)
* [Redis Hset 命令](http://www.runoob.com/redis/hashes-hset.html)
* [gopkg.in/redis.v3](https://godoc.org/gopkg.in/redis.v3)