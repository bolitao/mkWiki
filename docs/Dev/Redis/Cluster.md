## Cluster

``` shell
/app/soft/redis-5.0.5/bin/redis-cli -a powersi@redis --cluster create 172.18.20.116:6379 172.18.20.117:6379 172.18.20.118:6379 --cluster-replicas 0
```

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository@master/img/image-20200904102735186.png)