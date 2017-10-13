## RabbitMQ安装

### RabbitMQ 简介
AMQP，即 Advanced Message Queuing Protocol，高级消息队列协议是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP 的主要特征是面向消息、队列和路由，可靠且安全。RabbitMQ 是一个开源的 AMQP 实现，服务器端用 Erlang 语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP 等，支持 Ajax。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。RabbitMQ 中有一些概念需要我们在使用前先搞清楚，主要包括以下几个：Broker、Exchange、Queue、Binding、Routingkey、Producter、Consumer、Channel。

* 1.Broker 简单来说就是消息队列服务器的实体。
* 2.Exchange 接收消息，转发消息到绑定的队列上，指定消息按什么规则，路由到哪个队列。
* 3.Queue 消息队列载体，用来存储消息，相同属性的 queue 可以重复定义，每个消息都会被投入到一个或多个队列。
* 4.Binding 绑定，它的作用就是把 Exchange 和 Queue 按照路由规则绑定起来。
* 5.RoutingKey 路由关键字，Exchange 根据这个关键字进行消息投递。
* 6.Producter 消息生产者，产生消息的程序。
* 7.Consumer 消息消费者，接收消息的程序。
* 8.Channel 消息通道，在客户端的每个连接里可建立多个 Channel，每个 channel 代表一个会话。


### 安装 RabbitMQ 

RabbitMQ 是基于 Erlang 语言开发的，所以首先必须安装 Erlang 运行时环境。安装过程：
1. 下载 erlang-R15B-02.1.el5.x86_64.rpm 并安装

    ```sh
    # rpm -ivh erlang-R15B-02.1.el5.x86_64.rpm
    ```

2. 下载 rabbitmq-server-3.2.1-1.noarch.rpm 并安装

    ```sh
    # rpm -ivh rabbitmq-server-3.2.1-1.noarch.rpm
    ```

3. 启用相关插件

    ```sh
    # rabbitmq-plugins enable rabbitmq_management rabbitmq_web_stomp rabbitmq_stomp
    ```

4. 重启 RabbitMQ 服务

    ```sh
    # service rabbitmq-server restart
    ```


5. 验证是否安装成功
    此时我们可能通过 Web 浏览器来查看 RabbitMQ 的运行状态，浏览器中输入 `http://{server_ip}:15672`，用 guest/guest 默认的用户和密码登录后即可查看 RabbitMQ 的运行状态。



### 编译安装

#### Erlang安装

1. RabbitMQ是基于Erlang的，所以首先必须配置Erlang环境.
    从Erlang的官网 http://www.erlang.org/download.html 下载最新的erlang安装包，Linux和MacOSX下载的版本是 http://www.erlang.org/download.html
2. 然后解压下载的gz包tar  -xvf  *.tar.gz
3. cd 进入解压出来的文件夹
4. 执行 `./configure --prefix=/usr/local/erlang` 就会开始编译安装  会编译到 /usr/local/erlang 下 如果不报错就执行make 和 make install

    ```sh
    wget  http://erlang.org/download/otp_src_20.1.tar.gz
    tar  -xvf  otp_src_20.1.tar.gz
    cd otp_src_20.1
    ./configure --prefix=/usr/local/erlang
    make && make install
    ```

5. 修改/etc/profile文件，增加下面的环境变量

    ```sh
    ERL_HOME==/usr/local/erlang
    PATH=$ERL_HOME/bin:$PATH
    export ERL_HOME PATH
    # source /etc/profile //重加载配置文件
    ```

6. 测试erlang是否安装成

    ```sh
     /usr/local/erlang/bin/erl
    ```

7. 有几种退出ErlangShell的方法

    - 命令方式1：执行init:stop().   
    - 命令方式2：执行halt(). 
    - 快捷键方式1：Control+C然后选a
    - 快捷键方式2：Control+G然后按q


####  RabbitMQ安装
1. 下载 
    参考[Server Build Instructions](http://www.rabbitmq.com/build-server.html)
    [官方下载地址](http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.12/rabbitmq-server-generic-unix-3.6.12.tar.xz)

2. 安装  
    RabbitMQ3.6版本无需make、make install 解压就可以用

    ```sh
    wget "http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.12/rabbitmq-server-generic-unix-3.6.12.tar.xz"

    #解压rabbitmq，官方给的包是xz压缩包，所以需要使用xz命令
    xz -d rabbitmq-server-generic-unix-3.6.12.tar.xz

    #xz解压后得到.tar包，再用tar命令解压
    tar -xvf rabbitmq-server-generic-unix-3.6.12.tar

    #移动目录 看个人喜好
    cp -rf ./rabbitmq_server-3.6.1 /usr/local/
    cd /usr/local/

    #修改文件夹名
    mv rabbitmq_server-3.6.1 rabbitmq-3.6.1
    ln -s /usr/local/rabbitmq-3.6.1 /usr/local/rabbitmq

    #打开/etc/profile文件，在文件最后添如下两行环境变量 
    #set rabbitmq environment 
    #export PATH=$PATH:/usr/local/rabbitmq-3.6.1/sbin
    #使环境变量生效: 
    #source /etc/profile 

    #开启管理页面插件
    cd ./rabbitmq-3.6.1/sbin/
    ./rabbitmq-plugins enable rabbitmq_management
    ```

3. 启动：

    15672是rabbimq网页管理监听端口,5672是php客户端使用的端口，在浏览器中输入localhost:15672，可以看到如下页面：
    输入用户名guest和密码guest即可通过网页管理rabbitmq。

    ```sh
    #启动命令，该命令ctrl+c后会关闭服务
    cd /usr/local/rabbitmq/sbin
    ./rabbitmq-server

    #在后台启动Rabbit(可以实现后台运行)
    ./rabbitmq-server -detached

    #.查看启动是否成功: 
    netstat -tunlp | grep beam
     tcp 0 0 0.0.0.0:25672 0.0.0.0:* LISTEN 3308/beam.smp 
     tcp 0 0 0.0.0.0:15672 0.0.0.0:* LISTEN 3308/beam.smp 
     tcp 0 0 :::5672 :::* LISTEN 3308/beam.smp 

    #关闭服务
    ./rabbitmqctl stop

    #关闭服务(kill) 找到rabbitmq服务的pid   [不推荐]
    ps -ef | grep rabbitmq
    kill -9 [端口号]
    ```


#### 配置
主要参考官方文档 http://www.rabbitmq.com/configure.html
一般情况下，RabbitMQ的默认配置就足够了。如果希望特殊设置的话，有两个途径：
一个是环境变量的配置文件 `rabbitmq-env.conf` ；
一个是配置信息的配置文件 `rabbitmq.config`；
注意，这两个文件默认是没有的，如果需要必须自己创建。

1. rabbitmq-env.conf

    这个文件的位置是确定和不能改变的，位于：/etc/rabbitmq目录下（这个目录需要自己创建）。
    文件的内容包括了RabbitMQ的一些环境变量，常用的有：

    ```sh
    #RABBITMQ_NODE_PORT=    //端口号
    #HOSTNAME=
    RABBITMQ_NODENAME=mq
    RABBITMQ_CONFIG_FILE=        //配置文件的路径
    RABBITMQ_MNESIA_BASE=/rabbitmq/data        //需要使用的MNESIA数据库的路径
    RABBITMQ_LOG_BASE=/rabbitmq/log        //log的路径
    RABBITMQ_PLUGINS_DIR=/rabbitmq/plugins    //插件的路径
    ```

    具体的列表见：http://www.rabbitmq.com/configure.html#define-environment-variables

2. rabbitmq.config

    这是一个标准的erlang配置文件。它必须符合erlang配置文件的标准。
    它既有默认的目录，也可以在rabbitmq-env.conf文件中配置。
    文件的内容详见：http://www.rabbitmq.com/configure.html#config-items


#### 监控

主要参考官方文档：http://www.rabbitmq.com/management.html
RabbitMQ提供了一个web的监控页面系统，这个系统是以Plugin的方式进行调用的。
首先，在rabbitmq-env.conf中配置好plugins目录的位置：RABBITMQ_CONFIG_FILE
将监控页面所需要的plugin下载到plugins目录下，这些plugin包括：
-  mochiweb
-  webmachine
-  rabbitmq_mochiweb
-  amqp_client
-  rabbitmq_management_agent
-  rabbitmq_management

下载路径位于：http://www.rabbitmq.com/plugins.html#rabbitmq_management 



#### 参考

* [RabbitMQ的安装，配置，监控](http://blog.csdn.net/historyasamirror/article/details/6827870)
