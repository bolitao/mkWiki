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

此方法缺陷：不能应对 MQ 发送成功，但邮件服务出现 exception 的情况（TODO）

1. rabbitTemplate 设置

    在确认 MQ 发送成功时，修改 status 为成功状态。

    ``` java
    @Bean
    RabbitTemplate rabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(cachingConnectionFactory);
        rabbitTemplate.setConfirmCallback((data, ack, cause) -> {
            LOGGER.info("{}, {}, {}", data, ack, cause);
            String msgId = data.getId();
            LOGGER.warn(msgId);
            if (ack) { // 消息发送成功
                LOGGER.info(msgId + " 消息发送成功[MQ Config]");
                mailSendLogService.updateMailSendLogStatus(msgId, 1); // 修改记录表中为成功
            } else {
                LOGGER.error(msgId + " 消息发送失败[MQ Config] ack 异常");
            }
        });
        // TODO
        rabbitTemplate.setReturnCallback((msg, repCode, repText, exchange, routingKey) -> {
            LOGGER.error("消息发送失败[MQ Config 回调]");
        });
        return rabbitTemplate;
    }
    ```

1. MQ Listener

    在 MQ 消费者收到消息后，调用 MailService 发送邮件:

    ``` java
    @RabbitListener(queues = {MailConstants.MAIL_QUEUE_NAME})
    public void handler(Message message, Channel channel) {
        logger.info("已收到 " + MailConstants.MAIL_QUEUE_NAME + " Queue 的消息，开始邮件发送");
        Integer empId = (Integer) message.getPayload();
        MessageHeaders headers = message.getHeaders();
        Long tag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
        String msgId = (String) headers.get("spring_returned_message_correlation");
        if (redisTemplate.opsForHash().entries("mail_log").containsKey(msgId)) {
            //redis 中包含该 key，说明该消息已经被消费过
            logger.info(msgId + ":消息已经被消费");
            try {
                channel.basicAck(tag, false);//确认消息已消费
            } catch (IOException e) {
                logger.error("ack 异常: ", e);
            }
            return;
        }
        // 收到消息，发送邮件
        try {
            // 以下是构造 mail 相关 ，略去
            // ...
            mailSender.send(mail);
            redisTemplate.opsForHash().put("mail_log", msgId, "mail");
            channel.basicAck(tag, false);
            logger.info(msgId + ":邮件发送成功");
        } catch (Exception e) {
            try {
                channel.basicNack(tag, false, true);
            } catch (IOException ioException) {
                logger.error("nact 异常: ", ioException);
            }
            e.printStackTrace();
            logger.error("邮件发送失败：" + e.getMessage());
        }
    }
    ```

1. 重试机制

    其实就是定时任务，定时查询重试次数小于五次、状态为”投递中“的消息，查到的话就将 ID 通过 MQ 重新投递，以触发 MQ Listener、调用 MailService 发送邮件

    ``` java
    @Scheduled(cron = "0/10 * * * * ?")
    public void mailResendTask() {
        List<MailSendLog> logs = mailSendLogService.getMailSendLogByStatus();

        logs.forEach(mailSendLog -> {
            if (mailSendLog.getCount() >= 5) {
                mailSendLogService.updateMailSendLogStatus(mailSendLog.getMsgid(), 2);
            } else {
                mailSendLogService.updateCount(mailSendLog.getMsgid(), new Date());
                Integer empId = mailSendLog.getEmpid();
                rabbitTemplate.convertAndSend(MailConstants.MAIL_EXCHANGE_NAME, MailConstants.MAIL_ROUTING_KEY_NAME, empId, new CorrelationData(mailSendLog.getMsgid()));
            }
        });
    }
    ```

1. 测试类

    ``` java
    public Integer addEmp(int empId) {
        String msgId = UUID.randomUUID().toString();

        MailSendLog mailSendLog = new MailSendLog();
        mailSendLog.setMsgid(msgId);
        mailSendLog.setCreatetime(new Date());
        mailSendLog.setExchange(MailConstants.MAIL_EXCHANGE_NAME);
        mailSendLog.setRoutekey(MailConstants.MAIL_ROUTING_KEY_NAME);
        mailSendLog.setEmpid(empId);
        mailSendLog.setCount(0);
        mailSendLog.setTrytime(new Date(System.currentTimeMillis() + 1000L * 60 * MailConstants.MSG_TIMEOUT));

        mailSendLogService.insert(mailSendLog);

        rabbitTemplate.convertAndSend(MailConstants.MAIL_EXCHANGE_NAME, MailConstants.MAIL_ROUTING_KEY_NAME, empId, new CorrelationData(msgId));

        return 0;
    }
    ```

    在这一步：

    1. 如果 MQ 运行异常，消息未正常投递（`convertAndSend` 出错），那么 rabbitTemplate 中的 `updateMailSendLogStatus` 不会运行，MQ Listener 接收不到消息，不会调用 MailService，此时消息状态不会更改，由重试 Task 进行邮件的重新投递。

    2. 如果消息投递正常（RabbitMQ 工作正常），那么 Listener 会收到消息，调用 MailService 发送邮件；rabbitTemplate 中的回调会更新邮件状态为 1（已成功投递）

## TODO <!--TODO-->

- 高可用
- Demo 2 中 mailService 异常的重试机制
- 幂等
