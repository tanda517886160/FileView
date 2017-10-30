## 发布订阅

### 交换器

从Producer接收Message，然后投递到queue中。Exchange需要知道如何处理Message，是把Message放到特定queue中，还是放到多个queue中？或者丢弃.这个rule是通过Exchange 的type定义的。

![img](https://tanda517886160.github.io/resources/images/rabbitmq/5-1.png)

可用的交换类型: "direct", "topic","headers"和"fanout"
fanout exchange非常简单。您可能会猜测的名字,fanout就是广播模式,广播所有的消息到它知道所有队列。


### 临时队列

为了所有日志消息Rabbit需要设置一个新的、空的队列。服务器支持选择一个随机队列名称


# 捆绑

将消费者队列绑定到指定的交换机上， 交换将消息通过指定的交换机来发送到队列。

![img](https://tanda517886160.github.io/resources/images/rabbitmq/5-2.png) 


### 发布订阅模式

#### 1. 发布者 publisher.php

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明队列名
$exchange  = 'logs';

/*
    name: $exchange 交换机名称
    type: fanout 使用广播类型消息
    passive: false // 不核对交换机是否存在
    durable: false // 服务重启后交换机不存在
    auto_delete: true //信道关闭后交换机同步清楚
*/
$channel->exchange_declare($exchange, 'fanout', false, false, false);

//实例化一个消息
$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "Hello World!";
$msg = new AMQPMessage($data);

//发布消息，
$channel->basic_publish($msg, 'logs');
echo " [x] Sent ", $data, "\n";

// 最后关闭通道和连接
$channel->close();
$connection->close();
```


#### 1. 订阅者 subscriber.php

```php
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明交换机
$exchange  = 'logs';

/*
    name: $exchange 交换机名称
    type: fanout 使用广播类型消息
    passive: false // 不核对交换机是否存在
    durable: false // 服务重启后交换机不存在
    auto_delete: true //信道关闭后交换机同步清楚
*/
$channel->exchange_declare($exchange, 'fanout', false, false, false);

/*
    name: $queue    //队列名称， 为空时使用默认队列，类似像amq.gen-JzTY20BRgKO-HjmUJj0wLg。
    passive: false  //不检查队列是否存在
    durable: false  //服务器重启后队列不复存在
    exclusive: true //队列能不被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

//将队列绑定到指定交换机
$channel->queue_bind($queue_name, $exchange);

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

/**
 * 回调函数
 * @param  string $msg 消息体
 * @return void
 */
$callback = function($msg){
  echo ' [x] ', $msg->body, "\n";
};

/*
    消费消息
    queue: 制定队列
    consumer_tag: Consumer identifier
    no_local: Don't receive messages published by this consumer.
    no_ack: 服务器是否消息确认,默认为true是关闭的
    exclusive: 独占该消息，只有该channel才能消费这条消息
    nowait:
    callback: 回调函数
*/
$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

// 循环监听回调
while(count($channel->callbacks)) {
    $channel->wait();
}

// 最后关闭通道和连接
$channel->close();
$connection->close();
```
