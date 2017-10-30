
## 用户角色分类：
none、management、policymaker、monitoring、administrator

## 用户角色描述：
1. none
    * 不能访问 management plugin

2. management
    * 用户可以通过AMQP做的任何事外加：
    * 列出自己可以通过AMQP登入的virtual hosts
    * 查看自己的virtual hosts中的queues, exchanges 和 bindings
    * 查看和关闭自己的channels 和 connections
    * 查看有关自己的virtual hosts的“全局”的统计信息，包含其他用户在这些virtual hosts中的活动。

3. policymaker
    * management可以做的任何事外加：
    * 查看、创建和删除自己的virtual hosts所属的policies和parameters

4. monitoring
    * management可以做的任何事外加：
    * 列出所有virtual hosts，包括他们不能登录的virtual hosts
    * 查看其他用户的connections和channels
    * 查看节点级别的数据如clustering和memory使用情况
    * 查看真正的关于所有virtual hosts的全局的统计信息

5. administrator
    * policymaker和monitoring可以做的任何事外加:
    * 创建和删除virtual hosts
    * 查看、创建和删除users
    * 查看创建和删除permissions
    * 关闭其他用户的connections

### 添加账号
```
[root@centos19118:~]#  rabbitmqctl add_user  user_admin  'mypasswd'
Creating user "user_admin"

[root@centos19118:~]#  rabbitmqctl add_user  user_monitoring  'mypasswd'
[root@centos19118:~]#  rabbitmqctl  add_user  user_proj  'mypasswd'
```


### 设置权限
```
#管理员权限
[root@centos19118:~]# rabbitmqctl set_user_tags user_admin administrator
Setting tags for user "user_admin" to [administrator]
#监控权限
[root@centos19118:~]# rabbitmqctl set_user_tags user_monitoring monitoring
#专用项目权限
[root@centos19118:~]# rabbitmqctl set_user_tags user_proj management
```

### 查看并确认
```
[root@centos19118:~]# rabbitmqctl list_users
Listing users
user_admin [administrator]
user_monitoring   [monitoring]
user_proj   [management]
```


### 修改密码
```
[root@centos19118:/data/www/RedeemTask]# rabbitmqctl change_password user_admin 123456
Changing password for user "user_admin"
```
###


--------------
##  权限控制：

默认virtual host："/"
默认用户：guest
* guest具有"/"上的全部权限，仅能有localhost访问RabbitMQ包括Plugin，建议删除或更改密码。
* 可通过将配置文件中loopback_users置空来取消其本地访问的限制：
[{rabbit, [{loopback_users, []}]}]

用户仅能对其所能访问的virtual hosts中的资源进行操作。
这里的资源指的是virtual hosts中的exchanges、queues等。
操作包括对资源进行配置、写、读。
配置权限可创建、删除、资源并修改资源的行为，写权限可向资源发送消息，读权限从资源获取消息。
比如：

exchange和queue的declare与delete分别需要exchange和queue上的配置权限
exchange的bind与unbind需要exchange的读写权限
queue的bind与unbind需要queue写权限和exchange的读权限
发消息(publish)需exchange的写权限
获取或清除(get、consume、purge)消息需queue的读权限


对何种资源具有配置、写、读的权限通过正则表达式来匹配，具体命令如下：
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
其中，<conf> <write> <read>的位置分别用正则表达式来匹配特定的资源，如'^(amq\.gen.*|amq\.default)$'可以匹配server生成的和默认的exchange，'^$'不匹配任何资源

### 添加vhost
```sh
[root@centos19118:~]# rabbitmqctl  add_vhost /tan
Creating vhost "/tan"
```


### 设置权限：
如下命令使用户user_admin具有/vhost1这个virtual host中所有资源的配置、写、读权限以便管理其中的资源:
```
[root@centos19118:~]# rabbitmqctl  set_permissions -p /tan  user_admin '.*' '.*' '.*'
Setting permissions for user "tanda" in vhost "/tan"
```

### 查看权限：
```
[root@centos19118:~]# rabbitmqctl list_user_permissions user_admin
Listing permissions for user "user_admin"
/tan    .*  .*  .*
```


* 需要注意的是RabbitMQ会缓存每个connection或channel的权限验证结果、因此权限发生变化后需要重连才能生效。


