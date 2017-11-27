## RabbitMQ消息确认机制


### 一、两种消息确认机制

RabbitMQ提供了transaction、confirm两种消息确认机制。

* transaction 即事务机制，手动提交和回滚；
* confirm 该机制提供了Confirmlistener和waitForConfirms两种方式。

confirm机制效率明显会高于transaction机制，但后者的优势在于强一致性。如果没有特别的要求，建议使用conrim机制。
消息的确认机制只是确认发布者(publisher)发送消息到消息代理(broker)，由broker进行应答，不能确认消息是否有效消费。
为了确认消息是否被发送给queue，应该在发送消息中启用参数mandatory=true，使用ReturnListener接收未被发送成功的消息。


### 二、transaction机制
RabbitMQ中与事务机制有关的方法有三个：txSelect(), txCommit()以及txRollback(), txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务。
需要手动提交和回滚，txSelect用于将当前channel设置成transaction模式，执行txCommit，消息才会转发给队列进入ready状态；执行txRollback，消息被取消。如果txCommit提交成功了，则消息一定到达了broker了，，如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候便可以捕获异常通过txRollback回滚事务了。

事务的多了四个步骤：

* client发送Tx.Select
* broker发送Tx.Select-Ok(之后publish)
* client发送Tx.Commit
* broker发送Tx.Commit-Ok


PHP代码：

```php
$mandatory = true;
$immediate = true;
$channel->tx_select();
$channel->basic_publish($message, $exchange, $routing_key, $mandatory，$immediate);
$channel->tx_commit();
```


事务回滚是什么样子的。关键代码如下:

```php
try {
    $channel->tx_select();
    $channel->basic_publish($message, $exchange, $routing_key, $mandatory，$immediate);
    $result = 1 / 0; //制造异常
    $channel->tx_commit();
} catch (\Exception $e) {
    print_r($e->getMessage());
    $channel->tx_rollback();
}
```

### 三、Confirm模式

transaction机制中，采用事务机制实现降低RabbitMQ的消息吞吐量。
confirm模式中，生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID）,这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

简单流程：

1、确认publisher发送消息到broker，由broker进行应答(不能确认是否被有效消费)
2、confirmSelect，进入confirm消息确认模式，确认方式：1、异步ConfirmListener；2、同步waitForConfirms
3、ConfirmListener、waitForConfirms均需要配合confirm机制使用
4、basicNack、basicReject：参数requeue=true时，消息会重新进入队列
5、autoDelete队列在消费者关闭后不管是否还有未处理的消息都会关闭掉


```php
$channel->set_ack_handler(
    function (AMQPMessage $message) {
        echo "Message acked with content " . $message->body . PHP_EOL;
    }
);
$channel->set_nack_handler(
    function (AMQPMessage $message) {
        echo "Message nacked with content " . $message->body . PHP_EOL;
    }
);

/**
 * 开启确认模式
 * true-同步模式，消费者每应答一个消息，才发送下一个消息， false-异步模式
 */
$channel->confirm_select(false); //true 同步, false-异步

/*
    name: $exchange 交换机
    type: fanout  交换模式：扇形
    passive: false // don't check if an exchange with the same name exists
    durable: false // true-持久化
    auto_delete: true //true-信道关闭时自动删除
*/
$channel->exchange_declare($exchange, 'fanout', false, false, true);

$i = 1;
$msg = 'message index: ' + $i;
$msg = new AMQPMessage($msg, array('content_type' => 'text/plain'));
$channel->basic_publish($msg, $exchange);

/*
 * 等待来自消费者的应答
 */
$channel->wait_for_pending_acks();

while ($i <= 11) {
    $msg = new AMQPMessage($i++, array('content_type' => 'text/plain'));
    $channel->basic_publish($msg, $exchange);
}
$channel->wait_for_pending_acks();

/**
 * @param \PhpAmqpLib\Channel\AMQPChannel $channel
 * @param \PhpAmqpLib\Connection\AbstractConnection $connection
 */
function shutdown($channel, $connection)
{
    $channel->close();
    $connection->close();
}
register_shutdown_function('shutdown', $channel, $connection);
```

confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息，在channel被设置成confirm模式之后，所有被 publish 的后续消息都将被confirm(即ack)或者被nack一次。
但是没有对消息被confirm的快慢做任何保证，并且同一条消息不会既被 confirm又被nack 。
