## 变量概念：
* Nginx 配置中，变量只能存放一种类型的值，那就是字符串
* 标准ngx_rewrite模块的set方法可进行赋值 如： set $a  "Hello World" 
* Nginx 变量的创建只能发生在 Nginx 配置加载的时候，或者说 Nginx启动的时候,而赋值操作则只会发生在请求实际处理的时候
* 不创建而直接使用变量会导致启动失败,无法在请求处理时动态地创建新的 Nginx 变量
* Nginx 变量名的可见范围虽然是整个配置，但每个请求都有所有变量的独立副本，或者说都有各变量用来存放值的容器的独立副本，彼此互不干扰
* 一个请求在其处理过程中，即使经历多个不同的 location 配置块，它使用的还是同一套 Nginx 变量的副本


## Nginx 内建变量(预定义变量)

### ngx_http_core

      $uri           -- 当前请求的 URI,不含请求参数
      $request_uri   -- 最原始的 URI, 包含请求参数
      $arg_XXX       -- URL参数变量群, 当前请求名为 name 的 URI 参数的值，系统自动小写
      $cookie_XXX    -- COOKIE参数变量群
      $http_XXX      -- HTTP请求头变量群
      $sent_http_XXX -- HTTP响应头变量群
      $args          -- 请求参数串


* 取处理程序(get handler) ： 读取变量时执行的代码
* 存处理程序(set handler) ： 改写变量时执行的代码
* 不是所有的 Nginx 变量都拥有存放值的容器。拥有值容器的变量在 Nginx 核心中被称为“被索引的”（indexed）；反之，则被称为“未索引的”（non-indexed）
* 变量群一般都是未索引的，即没预先赋值，每次都是要的时候才去扫描解析

> ** Nginx内建变量大部分不允许修改或重新赋值 **


### ngx_set_misc 

如果你想对 URI 参数值中的 %XX 这样的编码序列进行解码，可以使用第三方 ngx_set_misc 模块提供的 set_unescape_uri 配置指令：

```
    location /test {
        set_unescape_uri $name $arg_name;
        set_unescape_uri $class $arg_class;

        echo "name: $name";
        echo "class: $class";
    }
```




### ngx_map (映射模块）

* map 指令是在 server 配置块之外，也就是在最外围的 http 配置块中定义的

* 解释：
  - 函数记法 y = f(x) 来说， $args 就是“自变量” x，而 $foo 则是“因变量” y
  - $foo 的值是由 $args 的值来决定的，将 $args 变量的值映射到了 $foo 变量上。
  - 完整的映射规则：当 $args 的值等于 debug 的时候，$foo 变量的值就是 1，否则 $foo 的值就为 0.

```
map $args $foo {
   default 0; #当其他条件都不匹配的时候，这个条件才匹配
   debug   1; #如果“自变量” $args 精确匹配了 debug
              #这个字符串，则把“因变量” $foo 映射到值        
}

server {
    listen 8080;

    location /test {
        set $orig_foo $foo;
        set $args debug;

        echo "original foo: $orig_foo";
        echo "foo: $foo";
    }
}

#结果
$ curl 'http://localhost:8080/test'
original foo: 0
foo: 0

# 第一行输出指示 $orig_foo 的值为 0,请求并没有提供 URL 参数串
# $foo 变量在第一次读取时，根据映射规则计算出的值被缓存住了, 
```


### Nginx变量的值两种特殊的值

*  一种是“不合法”（invalid），另一种是“没找到”（not found）。
*  由 set 指令创建的变量未初始化就用在“变量插值”中时,效果等同于空字符串，但那是因为 set 指令为它创建的变量自动注册了一个“取处理程序”，将“不合法”的变量值转换为空字符串

* 内建变量 $arg_XXX 在请求 URL 参数 XXX 并不存在时会返回特殊值“找不到”,但遗憾的是在 Nginx 原生配置语言中是不能很方便地把它和空字符串区分开来的, 只有通过第三方模块 ngx_lua

```
location /test {
    content_by_lua '
        if ngx.var.arg_name == nil then
            ngx.say("name: missing")
        else
            ngx.say("name: [", ngx.var.arg_name, "]")
        end
    ';
}
# ngx.say 这个 Lua 函数，也是 ngx_lua 模块提供的，功能上等价于 ngx_echo 模块的 echo 配置指令

#结果
$ curl 'http://localhost:8080/test'
name: missing

$ curl 'http://localhost:8080/test?name='
name: []
```

```
location /foo {
    content_by_lua '
        if ngx.var.foo == nil then
            ngx.say("$foo is nil")
        else
            ngx.say("$foo = [", ngx.var.foo, "]")
        end
    ';
}

location /bar {
    set $foo 32;
    echo "foo = [$foo]";
}

#结果
$ curl 'http://localhost:8080/foo'
$foo = []
#因为nginx在启动阶段已经给$foo初始化了，只是没赋值，赋值发生在调用阶段的容器副本
```


###  Nginx 变量也能存放数组类型的值

这种类型的复杂任务通过 ngx_lua 实现
```
location /test {
    array_split "," $arg_names to=$array;
    array_map "[$array_it]" $array;
    array_join " " $array to=$res;

    echo $res;
}
 ```
