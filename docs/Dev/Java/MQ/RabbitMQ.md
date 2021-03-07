# RabbitMQ 相关

## 安装

手动安装麻烦，要装 erlang、rabbitmq-admin，还要配置插件。

直接用 [docker-compose](https://docs.bolitao.xyz/Dev/Docker/compose%20conf/rabbitmq/) 吧！

## 相关知识

- [RabbitMQ Tutorials — RabbitMQ](https://www.rabbitmq.com/getstarted.html)

### RabbitMQ 模式与 AMQP 标准概念的对应

1. Publish/Subscribe 对应 fanout

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307121741.png)

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307121717.png)

2. Routing 对应 direct

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307122154.png)

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307121830.png)

1. topic 对应 topic

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307122222.png)

    ![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/20210307121843.png)

## demo

### demo1

[三种常用模式的简单示例](https://github.com/bolitao/simpmqdemo)

### demo2

[使用 RabbitMQ 进行邮件发送和重试](https://github.com/bolitao/mqdemo)

## TODO

- 高可用 <!--TODO-->
