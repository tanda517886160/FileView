## 工作队列


利用轮循分配来消费任务信息(竞争消费者模式)

![img](https://tanda517886160.github.io/resources/images/rabbitmq/4-1.png)

背后的主要思想工作队列(又名:任务队列)是为了避免立即做一个资源密集型任务,不得不等待它完成。相反,我们安排以后的任务要做。我们封装任务作为消息并将其发送到一个队列。一个工作进程在后台运行将流行的任务和最终执行这项工作。当您运行许多消费者的任务将在他们之间共享。


### 循环调度与公平的分配
使用一个任务队列的优点之一是能够轻易并行化"parallelise"工作。如果建立一个任务队列并添加多个消费者,,RabbitMQ将发送每个消息到下一个消费者,在序列中平均每个消费者将获得相同数量的信息。这种分发消息方式称为循环。

### 消息答复
为了确保消息不会丢失,RabbitMQ支持消息应答。发送ack(nowledgement)消费者告诉RabbitMQ特定的消息已经收到,RabbitMQ可以处理删除它。消息确认默认是关闭的。可以把他们的第四个参数设置为basic_consume为false(true意味着没有ack)消费队列一旦完成一个任务就发送适当的确认应答。


### 消息的持久性
知道如何确保即使消费者死亡任务也不会丢失。但是RabbitMQ服务器挂了的话任务消息仍旧会丢失。


### 任务队列和消费队列

#### 1. 任务队列 tasker.php
```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明队列名
$queue  = 'task_queue';

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue, false, true, false, false);

//实例化一个消息,并设置消息持久化
$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "Hello World!";
$msg = new AMQPMessage($data,array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT));

$channel->basic_publish($msg, '', $queue);

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();
```

#### 1. 工作队列 worker.php

![img](https://tanda517886160.github.io/resources/images/rabbitmq/4-2.png)

```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明队列名
$queue  = 'task_queue';

/*
    name: 队列名称
    passive: false
    durable: true //服务器重启后队列依旧存活
    exclusive: false //队列能被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare('task_queue', false, true, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo " [x] Received ", $msg->body, "\n";
  sleep(substr_count($msg->body, '.'));
  echo " [x] Done", "\n";
  $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};

/**
 * prefetch_count = 1设置。这告诉RabbitMQ不同时给多个消息到同一个消费者
 * 也就是说 在消息未处理完成前不分配新的任务给消费者
 */
$channel->basic_qos(null, 1, null);

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
$channel->basic_consume($queue, '', false, false, false, false, $callback);

// 循环监听回调
while(count($channel->callbacks)) {
    $channel->wait();
}

// 最后关闭通道和连接
$channel->close();
$connection->close();
```


在命令行(终端)运行3个脚本

同时开两个终端，运行：
```sh
php receive.php
```

运行发送消息脚本
```sh
php send.php
```
