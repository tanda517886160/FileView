## 单发单接模式

在这部分的教程中我们将用PHP编写两个程序,发送一个消息的生产者,消费者接收信息并打印出来。我们会掩盖一些细节的php-amqplib API,把精力集中在这个非常简单的事情开始。这是一个“Hello World”的消息。

在下面的图中,“P”是我们的生产和“C”是我们消费者。中间的框是一个队列,消息缓冲RabbitMQ代表消费者。

![img](https://tanda517886160.github.io/resources/images/rabbitmq/3-1.png)



### 第一步 配置环境

####  1.Composer安装php-amqplib

链接 ： https://github.com/php-amqplib/php-amqplib
在composer.json 中增加

```json
{
  "require": {
      "php-amqplib/php-amqplib": "2.7.*"
  }
}
```

并执行 `php composer.phar install`


### 第二步  加载配置

配置文件 config.php:
```php
require_once __DIR__.'/vendor/autoload.php';

define('HOST', 'localhost'); //RabbitMQ服务器主机IP地址
define('PORT', 5672); //RabbitMQ服务器端口
define('USER', 'guest'); // 连接RabbitMQ服务器的用户名,默认是guest
define('PASS', 'guest'); // 连接RabbitMQ服务器的用户密码
define('VHOST', '/'); //连接RabbitMQ服务器的vhost（服务器可以有多个vhost，虚拟主机，类似nginx的vhost）

//开启debug模式
define('AMQP_DEBUG', true);
```

### 第三步 发送者和接收者

我们叫消息发布者(sender)"send.php",消息接受者叫"receive.php".生产者蒋连接到RabbitMQ发送一条消息并退出
在"send.php"，我们必须包含必须的库和必要的类

![img](https://tanda517886160.github.io/resources/images/rabbitmq/3-2.png)

#### 1. 生产者 send.php：

```php
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中 
$channel = $connection->channel();
//声明队列名
$queue  = 'hello';

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue, false, false, false, false);

//实例化一个消息,并设置消息持久化
$messageBody = implode(' ', array_slice($argv, 1));
$message     = new AMQPMessage($messageBody, array('content_type' => 'text/plain', 'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT));

//发送消息到默认交换机
$channel->basic_publish($message, '', $queue);
echo " [x] Sent '{$message}'\n";

// 最后关闭通道和连接
$channel->close();
$connection->close();
```


#### 2. 接收者 receiver.php

从RabbitMQ接受消息,不同的生成者发送消息,并保持运行监听消息并打印出来。

![img](https://tanda517886160.github.io/resources/images/rabbitmq/3-3.png)

```php
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中 
$channel = $connection->channel();
//声明队列名
$queue  = 'hello';

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue, false, false, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";


/**
 * 回调函数
 * 
 * @param \PhpAmqpLib\Channel\AMQPChannel $channel
 * @param \PhpAmqpLib\Connection\AbstractConnection $connection
 */
$callback = function($msg) {
  echo " [x] Received ", $msg->body, "\n";
};

/*
    消费消息
    queue: 制定队列
    consumer_tag: Consumer identifier
    no_local: Don't receive messages published by this consumer.
    no_ack: 服务器是否核对确认消息
    exclusive: 独占该消息，只有该channel才能消费这条消息
    nowait:
    callback: 回调函数
*/
$channel->basic_consume($queue, '', false, true, false, false, $callback);

// 循环监听回调
while(count($channel->callbacks)) {
    $channel->wait();
}
```


在命令行(终端)运行2个脚本

```sh
php receive.php
```

```sh
php send.php
```

消费者将打印消息从发送方通过RabbitMQ。接收者将继续运行,等待消息(使用ctrl - c来阻止它),所以尝试发送方从另一个终端上运行。
