Redis 缓存了多少个 KEY
=======================
#### `INFO` 命令
```
127.0.0.1:6379> INFO
...
...
...
# Keyspace
db0:keys=1920216,expires=0,avg_ttl=0
```
查看 `# Keyspace` 区域，可知道当前 `key` 的情况 

或者直接
```
127.0.0.1:6379> INFO keyspace
# Keyspace
db0:keys=2652150,expires=0,avg_ttl=0

```

#### Bash
```
[cryven@codcodcog redis-4.0.6]$ src/redis-cli KEYS "*" | wc -l
2275155
```

#### `DBSIZE` 命令
```
127.0.0.1:6379> DBSIZE
(integer) 2459348
```
