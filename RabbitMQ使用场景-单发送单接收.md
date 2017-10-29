## 单发单接模式

### 环境

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


### 加载config

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


####  1.单发送单接(多接)收模式

1. 发布者 publisher_mode_1.php：
```php
require_once __DIR__.'/config.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$queue      = 'msgs';
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
$channel    = $connection->channel();

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue, false, true, false, false);

//实例化一个消息,并设置消息持久化
$messageBody = implode(' ', array_slice($argv, 1));
$message     = new AMQPMessage($messageBody, array('content_type' => 'text/plain', 'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT));

//发送消息到默认交换机
$channel->basic_publish($message);
$channel->close();
$connection->close();
```


2. 订阅者：

consumer_mode_1.php

```php
require_once __DIR__.'/config.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$queue      = 'msgs';
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
$channel    = $connection->channel();

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue, false, true, false, false);

/**
 * 绑定交换机，当未显示绑定交换机时，默认是绑定匿名交换机
 */
$channel->queue_bind($queue);


/**
 * @param \PhpAmqpLib\Message\AMQPMessage $message
 */
function process_message($message)
{
    echo "\n--------\n" . $message->body . "\n--------\n";
    $message->delivery_info['channel']->basic_ack($message->delivery_info['delivery_tag']);

    // 发送"quit"消息停止脚本
    if ($message->body === 'quit') {
        $message->delivery_info['channel']->basic_cancel($message->delivery_info['consumer_tag']);
    }
}

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
$channel->basic_consume($queue, $consumerTag, false, false, false, false, 'process_message');

/**
 * 回调函数
 * 
 * @param \PhpAmqpLib\Channel\AMQPChannel $channel
 * @param \PhpAmqpLib\Connection\AbstractConnection $connection
 */
function shutdown($channel, $connection)
{
    $channel->close();
    $connection->close();
}

register_shutdown_function('shutdown', $channel, $connection);

// 循环监听回调
while (count($channel->callbacks)) {
    $channel->wait();
}
```


### 使用

打开两个执行窗口，并分别执行

窗口1:
```
php publisher_mode_1.php "Hello World"
php publisher_mode_1.php "Hello World"
php publisher_mode_1.php "quit"
```

窗口2：
```php
php consumer_mode_1.php
```
