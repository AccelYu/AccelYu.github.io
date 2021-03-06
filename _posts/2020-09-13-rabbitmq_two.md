---
layout: post
title: RabbitMQ（二），进阶
categories: [Python, 消息队列]
---

RabbitMQ的交换机由erlang实现，效率媲美socket

<!-- more -->
## 交换机模式
如果消息发送给未绑定队列的交换机，消息将丢失

### 订阅模式
```
# 生产者
# 创建交换机
channel.exchange_declare(exchange='hello2',  # 交换机名
                         exchange_type='fanout',  # 发布订阅
                         durable=True)  # 持久化交换机
msg = 'Hello World!'
channel.basic_publish(exchange='hello2',
                      routing_key='',
                      body=msg,
                      properties=pika.BasicProperties(
                          delivery_mode=2)  # 持久化消息
                      )
print('Sent %r' % msg)
connection.close()
```
```
# 消费者
channel.exchange_declare(exchange='hello2',
                         exchange_type='fanout',
                         durable=True)
# 创建一个队列，exclusive名字随机且唯一
result = channel.queue_declare("", durable=True, exclusive=True)
queue_name = result.method.queue
# 将队列与交换机绑定
channel.queue_bind(exchange='hello2', queue=queue_name)
channel.basic_consume(queue=queue_name,
                      auto_ack=False,
                      on_message_callback=callback)
print('Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

### 关键字
```
# 生产者
# 创建交换机
exchange_name = 'log1'
channel.exchange_declare(exchange=exchange_name,  # 交换机名
                         exchange_type='direct',  # 关键字
                         durable=True)  # 交换机持久化
msg = 'Hello World!'
channel.basic_publish(exchange=exchange_name,
                      routing_key='info',
                      body=msg,
                      properties=pika.BasicProperties(
                          delivery_mode=2)
                      )
print('Sent %r' % msg)
connection.close()
```
```
# 消费者
exchange_name = 'log1'
channel.exchange_declare(exchange=exchange_name,
                         exchange_type='direct',
                         durable=True)
# 创建一个队列，exclusive名字随机且唯一
result = channel.queue_declare("", durable=True, exclusive=True)
queue_name = result.method.queue
# 将队列与交换机绑定
channel.queue_bind(queue=queue_name, exchange=exchange_name, routing_key='info')
channel.queue_bind(queue=queue_name, exchange=exchange_name, routing_key='warning')
channel.queue_bind(queue=queue_name, exchange=exchange_name, routing_key='error')

channel.basic_consume(queue=queue_name,
                      auto_ack=False,
                      on_message_callback=callback)
print('Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

### 通配符
```
# 生产者
# 创建交换机
exchange_name = 'log2'
channel.exchange_declare(exchange=exchange_name,  # 交换机名
                         exchange_type='topic',
                         durable=True)  # 通配符
msg = 'Hello World!'
channel.basic_publish(exchange=exchange_name,
                      routing_key='europe.weather',
                      body=msg,
                      properties=pika.BasicProperties(
                          delivery_mode=2)
                      )
print('Sent %r' % msg)
connection.close()
```
```
# 消费者
exchange_name = 'log2'
channel.exchange_declare(exchange=exchange_name,
                         exchange_type='topic',
                         durable=True)
# 创建一个队列，exclusive名字随机且唯一
result = channel.queue_declare("", durable=True, exclusive=True)
queue_name = result.method.queue
# 将队列与交换机绑定
channel.queue_bind(queue=queue_name, exchange=exchange_name, routing_key='europe.#')

channel.basic_consume(queue=queue_name,
                      auto_ack=False,
                      on_message_callback=callback)
print('Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

## RPC(Remote Procedure Call)
### 核心思想
client(请求)->client stub(将请求、自身地址序列化，寻址发送)  
->server stub(接收，反序列化)->server(处理请求，返回结果)  
->server stub(将结果序列化，寻址发送)->client stub(接收，反序列化)  
->client(得到结果，进行下一步)

### 客户端
```
import pika
import uuid
import time


class Client:
    @staticmethod
    def connect():
        credentials = pika.PlainCredentials('guest', 'guest')
        parameters = pika.ConnectionParameters('192.168.141.129', port=5672, virtual_host='/', credentials=credentials)
        connection = pika.BlockingConnection(parameters)
        return connection

    def __init__(self):
        # 赋值变量，一个循环值
        self.response = None
        # 随机一次唯一的字符串
        self.corr_id = ''
        # 链接远程
        self.connection = self.connect()
        self.channel = self.connection.channel()
        # 声明唯一queue
        result = self.channel.queue_declare('', exclusive=True)
        # 随机取queue名字
        self.callback_queue = result.method.queue
        # 接收callback_queue内的消息，即服务端处理完的结果
        self.channel.basic_consume(queue=self.callback_queue,
                                   auto_ack=False,
                                   on_message_callback=self.on_response)

    def on_response(self, ch, method, props, body):
        if self.corr_id == props.correlation_id:
            self.response = body
            # 手动应答，实例并发时拿错数据再还回去
            ch.basic_ack(delivery_tag=method.delivery_tag)

    def call(self, msg):
        # 每一次call都生成一个唯一id
        self.corr_id = str(uuid.uuid4())
        self.channel.queue_declare(queue='rpc_queue')
        self.channel.basic_publish(exchange='',
                                   routing_key='rpc_queue',
                                   body=msg,
                                   properties=pika.BasicProperties(
                                       # 执行命令之后结果返回给callaback_queue中
                                       reply_to=self.callback_queue,
                                       # 生成UUID 发送给消费端
                                       correlation_id=self.corr_id,))

        while self.response is None:
            # 非阻塞版的start_consuming()
            self.connection.process_data_events()
            print("listen...")
            time.sleep(1)
        return self.response


if __name__ == '__main__':
    client_rpc = Client()
    response = client_rpc.call('ok')
    print("Got %r" % response)
```

### 服务端
```
import pika


def func(msg):
    return 'return ' + msg.decode()


def on_request(ch, method, props, body):
    print('Request content %r' % body)
    response = func(body)

    ch.basic_publish(exchange='',
                     # 生产端随机生成的queue
                     routing_key=props.reply_to,
                     body=response,
                     # 获取UUID唯一 字符串数值
                     properties=pika.BasicProperties(
                         correlation_id=props.correlation_id))


if __name__ == '__main__':
    credentials = pika.PlainCredentials('guest', 'guest')
    parameters = pika.ConnectionParameters('192.168.141.129', port=5672, virtual_host='/', credentials=credentials)
    connection = pika.BlockingConnection(parameters)
    channel = connection.channel()

    channel.queue_declare(queue='rpc_queue')
    channel.basic_consume(queue="rpc_queue",
                          auto_ack=True,
                          on_message_callback=on_request)
    print("Waiting RPC requests")
    channel.start_consuming()
```

## 集群
