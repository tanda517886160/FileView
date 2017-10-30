

## 生产者（Publisher）发布消息流程：

    1、打开RabbitMQ连接; 
    2、创建Channel通道； 
    3、声名一个exchange交换机； 
    4、生成一条消息； 
    5、发布消息； 
    6、关闭Channel通道； 
    7、关闭RabbitMQ连接。 
       

## 消费者(Consumer) 订阅和消费消息流程：

    1、打开RabbitMQ连接; 
    2、创建Channel通道； 
    3、声名一个exchange交换机； 
    4、声名一个queue队列； 
    5、将queue队列绑定到exchange交换机； 
    6、消费消息； 
    7、关闭Channel通道； 
    8、关闭RabbitMQ连接。 


## 案例：

1. producer_with_comfirms.py

```py
#!/usr/bin/env python
# -*- coding:utf-8 -*-
  
import pika,sys  
from pika import spec  
  
#在"/"虚拟主机vhost上通过用户guest建立channel通道  
user_name = 'guest'  
user_passwd = 'guest'  
target_host = 'localhost'  
vhost = '/'  
cred = pika.PlainCredentials(user_name,user_passwd)  
conn_params = pika.ConnectionParameters(target_host,  
                                        virtual_host = vhost,  
                                        credentials = cred)  
conn_broker = pika.BlockingConnection(conn_params)  
channel = conn_broker.channel()  
  
#定义消息发布后publisher接受到的确认信息处理函数  
def confirm_handler(frame):  
    if type(frame.method) == spec.Confirm.SelectOk:  
        """生产者创建的channel处于‘publisher comfirms’模式"""  
        print 'Channel in "confirm" mode!'  
    elif type(frame.method) == spec.Basic.Nack:  
        """生产者接受到消息发送失败并且消息丢失的消息"""  
        print 'Message lost!'  
    elif type(frame.method) == spec.Basic.ack:  
        if frame.method.delivery_tag in msg_ids:  
            """生产者接受到成功发布的消息"""  
            print 'Confirm received!'  
            msg_ids.remove(frame.method.delivery_tag)  
  
#将生产者创建的channel处于"publisher confirms"模式  
channel.confirm_delivery(callback = confirm_handler)  
  
#创建一个direct类型的、持久化的、没有consumer时队列是否自动删除的exchage交换机  
channel.exchange_declare(exchange = 'hello-exch',  
                              type = 'direct',  
                              passive = False,  
                              durable = True,  
                              auto_delete = False)  
#使用接收到的信息创建消息  
msg = sys.argv[1]  
msg_props = pika.BasicProperties()  
msg_props.content_type = 'text/plain'  

#持久化消息  
msg_props.delivery_mode = 2  
print '正在发布消息.......'  
#发布消息  
channel.basic_publish(body = msg,  
                      exchange = 'hello-exch',  
                      properties = msg_props,  
                      routing_key = 'hala')  
print '消息已发布完成.....'  
channel.close()  
conn_broker.close()  
```


2. consumer_with_ack.py

```py
#!/usr/bin/env python
# -*- coding:utf-8 -*- 
  
import pika  
  
#在"/"虚拟主机vhost上通过用户guest建立channel通道  
user_name = 'guest'  
user_passwd = 'guest'  
target_host = 'localhost'  
vhost = '/'  
cred = pika.PlainCredentials(user_name,user_passwd)  
conn_params = pika.ConnectionParameters(target_host,  
                                        virtual_host = vhost,  
                                        credentials = cred)  
conn_broker = pika.BlockingConnection(conn_params)  
conn_channel = conn_broker.channel()  
  
#创建一个direct类型的、持久化的、没有consumer时，队列是否自动删除exchage交换机  
conn_channel.exchange_declare(exchange = 'hello-exch',  
                              type = 'direct',  
                              passive = False,  
                              durable = True,  
                              auto_delete = False)  
  
#创建一个持久化的、没有consumer时队列是否自动删除的名为“hell-queue”  
conn_channel.queue_declare(queue = 'hello-queue',  
                           durable = True,  
                           auto_delete = False)  
  
#将“hello-queue”队列通过routing_key绑定到“hello-exch”交换机  
conn_channel.queue_bind(queue = 'hello-queue',  
                   exchange = 'hello-exch',  
                   routing_key = 'hala')  
  
#定义一个消息确认函数，消费者成功处理完消息后会给队列发送一个确认信息，然后该消息会被删除  
def ack_info_handler(channel,method,header,body):  
    """ack_info_handler """      
    print 'ack_info_handler() called!'  
    if body == 'quit':  
        channel.basic_cancel(consumer_tag = 'hello-hala')  
        channel.stop_sonsuming()  
    else:  
        print body  
        channel.basic_ack(delivery_tag = method.delivery_tag)  
  
conn_channel.basic_consume(ack_info_handler,  
                           queue = 'hello-queue',  
                           no_ack = False,  
                           consumer_tag = 'hello-hala')  
  
print 'ready to consume msg...'  
conn_channel.start_consuming()  
```

3. 运行

```sh
/usr/local/rabbitmq/sbin/rabbitmq-server start & 
python ./consumer_with_ack.py 
python ./producer_with_comfirms.py 'hello world' 
```
