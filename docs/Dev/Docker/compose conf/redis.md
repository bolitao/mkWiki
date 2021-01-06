``` yml
version: '3'
services:
  redis:
    image: redis
    container_name: docker_redis
    volumes:
      - /data/redis/datadir:/data/
      - /data/redis/redisconf:/usr/local/etc/redis/
      - /data/redis/logs:/logs/
    # command:
    #   - redis-server /usr/local/etc/redis/redis.conf
    ports:
      - 6379:6379
```

TODO:

- 主从
- 集群

> REF
> 1. [使用docker-compose配置redis服务 - 落叶&不随风 - 博客园](https://www.cnblogs.com/xpengp/p/12713374.html)
