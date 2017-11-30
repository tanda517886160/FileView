## Nginx 指令之location


### 指令：

* 语法规则： `location [=|~|~*|^~] /uri/ { … }``
* 规则解释：

    =  表示精确匹配
    ^~ 表示uri以某个常规字符串开头，理解为匹配 url路径即可。
    ~  表示区分大小写的正则匹配
    ~* 表示不区分大小写的正则匹配
    !~和!~* 分别为区分大小写不匹配及不区分大小写不匹配 的正则
    /  通用匹配，任何请求都会匹配到。
    @  定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
   


> nginx不对url做编码，因此请求为 `/static/20%/aa`，可以被规则 `^~ /static/ /aa` 匹配到（注意是空格）。
> 多个location配置的情况下匹配顺序为：
> 首先匹配 =，其次匹配^~, 其次是按文件中顺序的正则匹配，最后是交给 / 通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。



### 匹配的优先级

* 与location在配置文件中的顺序无关
* = 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配
* 普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
* ^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。
* 最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。


### location 优先级官方文档

```
1. Directives with the = prefix that match the query exactly. If found, searching stops.
2. All remaining directives with conventional strings, longest match first. If this match used the ^~ prefix, searching stops.
3. Regular expressions, in order of definition in the configuration file.
4. If #3 yielded a match, that result is used. Else the match from #2 is used.

1. =前缀的指令严格匹配这个查询。如果找到，停止搜索。
2. 所有剩下的常规字符串，最长的匹配。如果这个匹配使用^〜前缀，搜索停止。
3. 正则表达式，在配置文件中定义的顺序。
4. 如果第3条规则产生匹配的话，结果被使用。否则，使用第2条规则的结果。
```


### 示例
来源: http://www.nginx.cn/115.html

* 示例

```ngx
location  = / {
  # 只匹配"/".
  [ configuration A ] 
}
location  / {
  # 匹配任何请求，因为所有请求都是以"/"开始
  # 但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ] 
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ] 
}
location ~* .(gif|jpg|jpeg)$ {
  # 匹配以 gif, jpg, or jpeg结尾的请求. 
  # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
  [ configuration D ] 
}

```

* 请求URI

```
/ -> 符合configuration A
/documents/document.html -> 符合configuration B
/images/1.gif -> 符合configuration C
/documents/1.jpg ->符合 configuration D
```

*  `@location` 例子

```ngx
error_page 404 = @fetch;

location @fetch(
    proxy_pass http://fetch;
)
```
