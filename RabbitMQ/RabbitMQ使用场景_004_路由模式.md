## 路由模式

queue只感兴趣这个的exchange。只希望接收交换机中的关键信息，或者说指定内容的信息， 而忽略交换机中的其他消息。
direct exchange背后的路由算法很简单——即一个消息的队列binding_key完全匹配message的routing_key。

* 1. 交换机绑定不同key的队列：
![img](https://tanda517886160.github.io/resources/images/rabbitmq/6-1.png)


在上图中: exchange X和两个queue绑定在一起。queue Q1的binding key是orange。queue Q2的binding key是black和green。
    当P publish key是orange时，exchange会把它放到Q1。如果是black或者green那么就会到Q2。其余的Message都会被丢弃。

* 2. 交换机绑定多个相同key的队列：
![img](https://tanda517886160.github.io/resources/images/rabbitmq/6-2.png)

多个queue绑定到相同的binding_key是合法的。在例子中,可以添加一个绑定X和Q1之间绑定关键黑色。在这种情况下,直接交换将像扇出,广播到所有匹配的消息队列。消息路由关键黑色将Q1和Q2。


* 交换机绑定多个不同key到相同队列
![img](https://tanda517886160.github.io/resources/images/rabbitmq/6-3.png)


## 相关代码：

#### 1.publisher_direct.php

```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明交换机名称
$exchange  = 'direct_logs';
//路由键名称
$router_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$channel->exchange_declare($exchange, 'direct', false, false, false);

//消息
$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";
$message = new AMQPMessage($data);

//发送消息
$channel->basic_publish($message, $exchange, $router_key);
echo " [x] Sent ",$queue,':',$data," \n";

$channel->close();
$connection->close();
```

#### 2. subscriber_dircetor.php

```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明交换机名称
$exchange  = 'direct_logs';
//路由键名称
$router_keys = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';


$channel->exchange_declare($exchange, 'direct', false, false, false);

/*
    name: $queue    //队列名称， 为空时使用默认队列，类似像amq.gen-JzTY20BRgKO-HjmUJj0wLg。
    passive: false  //不检查队列是否存在
    durable: false  //服务器重启后队列不复存在
    exclusive: true //队列能不被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

//指定的路由键
$router_keys = array_slice($router_keys, 1);
if(empty($router_keys)) {
    file_put_contents('php://stderr', "Usage: $argv[0] [info] [warning] [error]\n");
    exit(1);
}
foreach($router_keys as $router_key) {
    $channel->queue_bind($queue_name, $exchange, $router_key);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
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
