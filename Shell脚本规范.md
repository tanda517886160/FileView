# Shell脚本编写规范


* 1 脚本名以.sh结尾，名称尽量见名之意，比如ClearLog.sh Clear_Log.sh clearlog.sh SerRestart.sh Ser_Restart.sh;
* 2 尽量使用UTF-8编码，注释及输出尽量使用英文；
* 3 一般给到执行权限，但一些关于变量的配置文件不用加执行权限；
* 4 执行的时候可以使用bash 执行，或者使用bash -x执行，可以直观的显示具体的执行过程；
* 5 脚本首行使用/bin/bash,没有空格，不带任何选项；
* 6 第二行为空格，或者是添加一行空注释
* 7 接着开始注释内容：文件名、功能描述、作者、最后修改日期、版本号以及一些说明，还加上邮箱/手机号做为联系，如果可以，需要加上版权声明； 
* 8 注释内容之后空一行开始定义shell脚本中的变量；
* 9 脚本内的变量定义，尽量使用大写，或者大小写驼峰写法,或者使用下划线连接的方式。变量名要见名之意，避免a,b,c类似的定义，变量的定义前后不要用空格。
    - 如果是整形，需要使用declare -i来声明。
    - 如果是数组，则需要使用declare -a来声明。
    - 如果是只读变量，则需要使用declare -r来声明。
    - 变量值尽量使用双引号引起来，如果要使用强引用，如变量值中包含$符号，则使用”单引号引起来。
    - 如果要将命令的执行结果赋值给变量，则使用反引号，或者使用$().
* 10 变量的引用使用以下方式：
```
    ${GameZone}
    $GameZone
```
    推荐使用第一种，如：tar zcf ${GameZone}.tar.gz /apps/data/
* 11 单引号和双引号混合使用的场景：
```
     echo 'Welcome to "my school"'
```

* 12 在某些特殊的环境下，shell脚本里引用的命令，有可能是自己定义的bin路径，在执行的时候会报出command not found，
    解决的方式是在执行的时候命令跟全路径，或者在脚本的开始，显式的设置一下PATH 变量，
    如：
``` 
    export PATH=”/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin:/apps/bin/”
```

* 13 建议在脚本执行的开始重读下/etc/profile 或者是自己定义的关于环境变量的配置文件，推荐使用source， 如： 
```
    source /etc/profile
    source /opt/sh/appenv.sh
```

* 14 使用here document；
    如果脚本在执行的时候需要大段输出提示信息，可是使用以下方式：
```
    cat << EOF
    This scripts used for XXX
    Usage:$0 [option]
    Pls be careful.
    Enjoy Yourself.
    EOF
    如果只是单行提示信息，可是使用echo的方式，可以添加颜色：
    echo “Welcome to use my script”
```


* 15 如果需要在脚本里生成配置文件的模板，也可以使用here document的方式，示例如下：
```
   cat>>/etc/rsyncd.conf<<EOF
   log file = /usr/local/logs/rsyncd.log
   transfer logging = yes
   log format = %t %a %m %f %b
   syslog facility = local3
   timeout = 300
   [data1]
   path=/home/username
   list=yes
   ignore errors
   auth users = data1user
   secrets file=/etc/rsyncd/rsyncd.secrets
   comment = some description about this moudle
   exclude = test1/ test2/
   EOF
```

* 16 如果需要创建临时文件，可以使用如下方式：
```
       mktemp -d /tmp/file$$
```

* 17 条件测试的时候，尽量使用[[]],而不用[]或者test，因为[[]]功能会更强大
```                                             
       [[ -d /tmp/logs ]]
      不在使用[ “x$NAME” == “x” ]这种方式；
```

* 18 算数运算使用(())或者是中括号，但是记得括号里面的变量不要再加$
```
       ((12+i))
       而非((12+$i))
```

* 19 使用高级变量的用法，比如使用
```
       ${GameZone:?”Error Message”}确保关键变量已经定义
       ${GameZone:=”S1″} 或者设置默认值
       否则：
       rm -rf ${GameZone}/* 后果不堪设想
```

* 20 可以使用&& ||来替代简单的if-then-else-fi语句。
* 21 尽量给每条语句或者代码段的执行给一个执行结果状态，如果某条命令执行失败，则exit N.
       尽可能使用$?来检查前面一条命令的执行状态。
* 22 流程控制语句尽量使用一下方式：
```
       for I in {1..10};do
        ..。
       done
       while true;do
        …
       done
       if [];then
        …
       fi
```

* 23 如果命令过长，可以分成多行来写，比如：
```
       ./configure \
       –prefix=/usr \
       –sbin-path=/usr/sbin/nginx \
       –conf-path=/etc/nginx/nginx.conf \
       –error-log-path=/var/log/nginx/error.log \
       –http-log-path=/var/log/nginx/access.log \
       –pid-path=/var/run/nginx/nginx.pid  \
       –lock-path=/var/lock/nginx.lock \
```

* 24 shell脚本并不要求强制缩进，但是要养成缩进的好习惯，可以使用两个空格，建议使用tab键。如：
```
       if [];then
         …
       fi
```

* 25 尽可能多的注释信息。
* 26 想要获取当前脚本所在目录，可以使用
```
      ScriptDir=$(cd $(dirname $0) && pwd)
```
* 27 尽可能的使用函数的功能，将不同的功能定义为函数，直接引用函数；
* 28 如果自定义环境变量，可以专门写到一个文件中，避免在/etc/profile中添加；
* 29 禁止使用SUID和SGID以及ACL用户访问控制列表的功能，如果需要较高权限，可以使用sudo；
* 30 关键的操作须有日志输出，专门记录操作的成功或者失败以及执行的时间点。
* 31 脚本内可能包含敏感信息，如服务器密码或者是数据库密码，如果公开之前先确认敏感信息是否已经删除。