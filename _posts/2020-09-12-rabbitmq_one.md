---
layout: post
title: RabbitMQ（一），基础
categories: [Python, 消息队列]
---

RabbitMQ是一种最常用的异步消息队列

<!-- more -->
## 概念
### AMQP
producer->exchange->queue  

## 简单模式
### 生产者
```python
import pika

if __name__ == '__main__':
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters('192.168.141.129', port=5672, virtual_host='/', credentials=credentials)
    connection = pika.BlockingConnection(parameters)
    channel = connection.channel()
    # 创建队列，durable设置为可持久化
    channel.queue_declare(queue='hello1', durable=True)
    msg = 'Hello World!'
    channel.basic_publish(exchange='',  # 简单模式
                          routing_key='hello1',  # 指定队列
                          body=msg,  # 消息内容
                          properties=pika.BasicProperties(
                              delivery_mode=2)  # 此数据持久化
                          )
    print('Sent %r' % msg)
```

### 消费者
```python
import pika


def callback(ch, method, properties, body):
    print('%r:%r' % (method.routing_key, body))
    # 手动应答，应答完再出队，防止处理逻辑异常导致数据丢失
    ch.basic_ack(delivery_tag=method.delivery_tag)


if __name__ == '__main__':
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters('192.168.141.129', port=5672, virtual_host='/', credentials=credentials)
    connection = pika.BlockingConnection(parameters)
    channel = connection.channel()
    # 创建队列，防止消费者先于生产者启动报错
    channel.queue_declare(queue='hello1', durable=True)
    # 公平分发：第一轮按照连接顺序，第二轮先到先得（资源空闲）
    channel.basic_qos(prefetch_count=1)
    channel.basic_consume(queue='hello1',
                          auto_ack=False,  # 设置应答方式，False为手动应答
                          on_message_callback=callback)  # 回调函数
    print('Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()
```

## 消息的准确性
### 丢失
1. 原因：
   1. 生产者发送失败
   2. 消费者逻辑异常

2. 如何避免
   1. 将失败的消息存到redis，然后统一拉取，用定时任务重新发送（推荐celery）
   2. 手动应答

### 重复
1. 原因：
   1. 生产者发送成功，mq发送的确认生产者没收到，生产者又发一遍
   2. 消费者消费成功后发送的ack应答mq没收到，mq又发一遍

2. 如何避免
生产者生成唯一id拼接在消息中；消费者获取id，消费前验证id是否重复