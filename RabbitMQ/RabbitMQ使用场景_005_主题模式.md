## 主题模式


### Topic exchange

消息发送到topic exchange不可能任意routing_key——它必须是一个单词列表,使用点"."分隔。可以是任何单词,但通常他们指定一些功能连接到消息。一些有效routing_key例子:"stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit".routing key可以有尽可能多的单词,但是长度不能超过255字节。

binding_key必须在相同的形式。topic exchange背后的逻辑是类似于直接一个消息发送与特定routing_key交付的所有队列与匹配的binding_key绑定。

### binding_keys有两个重要的特殊情况:
> 1. 星号"*"完全可以代表任意一个单词.
> 2. #(hash)可以代表0个或者多个单词

请看下面一个例子:
![img](https://tanda517886160.github.io/resources/images/rabbitmq/6-1.png)

在这个例子中,我们将发送消息,所有描述动物.消息将发送到routing_key,由3个单词和2个点组成.第1个routing_key单词celerity(快速、敏捷),第2个color(颜色),第3个species(物种): "<celerity>.<colour>.<species>".


#### 我们创建了2个队列3个绑定: 
> 1. Q1 binding_key = "*.orange.*"
> - 队列Q1绑定了(orange)
> 2. Q2 binding_key = "*.*.rabbit" and ,
> - 队列Q2绑定了(rabbit)和(lazy)


####  绑定可以概括
> 1. 一个消息routing_key设置为"quick.orange.rabbit",它将被放到2个队列.
> 2. 消息 "lazy.orange.elephant"也会被放到2个队列,
> 3. 另一方面消息"quick.orange.fox" 会被放到Q1队列. 
> 4. 消息"lazy.brown.fox"会被放到Q2队列
> 5. 消息"lazy.pink.rabbit"虽然匹配了2个绑定,但是2个绑定都在Q2队列,只会放一次到Q2队列
> 6. 消息"quick.brown.fox" 不匹配任何绑定会被丢弃
> 7. 如果打破规则,发送一条消息,该消息带有一个或四个单词,如"orange"或"quick.orange.male.rabbit"? 这些消息不会匹配任何绑定会被丢弃。
> 8. 消息"lazy.orange.male.rabbit" 尽管它有四个单词,它能匹配最后一个绑定("lazy.#").它会被放到Q2队列

#### Topic exchange 特点
> 1. Topic exchange 是其他模式的升级版
> 2. 当binding_key使用"#",队列就会接受所有的消息,就行fanout exchange
> 3. 当binding_key不使用特殊字符("*"和 "#"),它就像direct exchange


### 示例(demo):
日志系统使用 topic exchange.将一个消费队列绑定两个路由键: "<facility>.<severity>".

1. emit_topic.php

```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明交换机名称
$exchange  = 'topic_logs';

$channel->exchange_declare($exchange, 'topic', false, false, false);

//路由键名称
$routing_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'anonymous.info';

//消息
$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";
$msg = new AMQPMessage($data);

//发布
$channel->basic_publish($msg, $exchange, $routing_key);
echo " [x] Sent ",$routing_key,':',$data," \n";

// 最后关闭通道和连接
$channel->close();
$connection->close();
```


2. receive_topic.php

```php
require_once __DIR__ . '/config.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

// 创建好服务器连接
$connection = new AMQPStreamConnection(HOST, PORT, USER, PASS, VHOST);
// 创建"channel"通道和声明队列名和发送消息到队列中
$channel = $connection->channel();
//声明交换机名称
$exchange  = 'topic_logs';
//队列名称
$queue_name  = 'queue_log_1';

$channel->exchange_declare($exchange, 'topic', false, false, false);

/*
    name: $queue    //队列名称， 为空时使用默认队列，类似像amq.gen-JzTY20BRgKO-HjmUJj0wLg。
    passive: false  //不检查队列是否存在
    durable: false  //服务器重启后队列不复存在
    exclusive: true //队列能不被其他channel访问
    auto_delete: false //channel关闭之后队列不删除
*/
$channel->queue_declare($queue_name, false, false, true, false);

//路由keys
$binding_keys = array_slice($argv, 1);
if( empty($binding_keys )) {
    file_put_contents('php://stderr', "Usage: $argv[0] [binding_key]\n");
    exit(1);
}

foreach($binding_keys as $binding_key) {
    $channel->queue_bind($queue_name, $exchange, $binding_key);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

/**
 * 回调函数
 * 
 * @param  {[type]} $msg [description]
 * @return {[type]}      [description]
 */
$callback = function($msg){
    echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

//开始消费
$channel->basic_consume($queue_name, '', false, true, false, false, $callback);


while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```
