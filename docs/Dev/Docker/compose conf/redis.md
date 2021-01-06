``` yml
version: '3'
services:
  redis:
    image: redis
    container_name: docker_redis
    volumes:
      - /data/redis/redisconf/redis.conf:/usr/local/etc/redis/redis.conf
      - /data/redis/datadir:/data/
      - /data/redis/logs:/logs/
    command:
      - /usr/local/etc/redis/redis.conf
    ports:
      - 6379:6379
    restart: always
```

## redis conf

``` shell
nano /data/redis/redisconf/redis.conf
```

conteng:

``` conf
requirepass bolitao

save 900 1
save 300 10
save 60 1000

appendonly yes
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

TODO:

- 主从
- 集群

> REF
> 1. [使用docker-compose配置redis服务 - 落叶&不随风 - 博客园](https://www.cnblogs.com/xpengp/p/12713374.html)
