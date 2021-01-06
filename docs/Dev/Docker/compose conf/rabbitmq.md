## conf

``` yml
services:
    rabbitmq:
        image: rabbitmq:latest
        volumes:
            - /data/rabbitmq/etc/:/etc/rabbitmq/
            - /data/rabbitmq/data/:/var/lib/rabbitmq/
            - /data/rabbitmq/logs/:/var/log/rabbitmq/
        environment:
            RABBITMQ_ERLANG_COOKIE: "SWQOKODSQALRPCLNMEQG"
            RABBITMQ_DEFAULT_USER: "rabbitmq"
            RABBITMQ_DEFAULT_PASS: "rabbitmq"
            RABBITMQ_DEFAULT_VHOST: "/"
        ports:
            - 5672:5672
            - 15672:15672
```

## 无法启动

``` shell
touch: cannot touch '/etc/rabbitmq/rabbitmq.conf': Permission denied
```

不使用 volume 挂载则无此问题。

``` shell
chown -R 999:999 /data/rabbitmq
```

降低权限，在我的机器上，`999:999` 的实质意义为 `openmediavault-webgui:openmediavault-config`。

## enable plugin

``` shell
docker exec -it rabbitmq_rabbitmq_1 bash
rabbitmq-plugins enable rabbitmq_management
```
