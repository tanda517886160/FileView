
# xpath路径表达式笔记


xpath就是选择XML文件中节点的方法, 所谓节点（node），就是XML文件的最小构成单位，一共分成7种。

    + element（元素节点）
    + attribute（属性节点）
    + text （文本节点）
    + namespace （名称空间节点）
    + processing-instruction （处理命令节点）
    + comment （注释节点）
    + root （根节点）

## 一、xpath表达式的基本格式

xpath通过"路径表达式"（Path Expression）来选择节点。在形式上，"路径表达式"与传统的文件系统非常类似。

    + 斜杠（/）作为路径内部的分割符。
    + 同一个节点有绝对路径和相对路径两种写法。
    + 绝对路径（absolute path）必须用"/"起首，后面紧跟根节点，比如/step/step/...。
    + 相对路径（relative path）则是除了绝对路径以外的其他写法，比如 step/step，也就是不使用"/"起首。
    + "."表示当前节点。
    + ".."表示当前节点的父节点


## 二、选择节点的基本规则

    + nodename（节点名称）：表示选择该节点的所有子节点
    + "/"：表示选择根节点
    + "//"：表示选择任意位置的某个节点
    + "@"： 表示选择某个属性


## 三、选择节点的实例

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <bookstore>
      <book>
        <title lang="eng">Harry Potter</title>
        <price>29.99</price>
      </book>
      <book>
        <title lang="eng">Learning XML</title>
        <price>39.95</price>
      </book>
    </bookstore>


在下面的表格中，我们已列出了一些路径表达式以及表达式的结果：

实例:

| 路径表达式      |  结果                                             |
| --------------- |:-------------------------------------------------:|
| bookstore       | 选取 bookstore 元素的所有子节点。                 |
| /bookstore      | 选取根元素 bookstore。 <br/>注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ |
| bookstore/book  | 选取属于 bookstore 的子元素的所有 book 元素。     |
| //book          | 选取所有 book 子元素，而不管它们在文档中的位置。  |
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。。                     |
| //@lang         | 选取名为 lang 的所有属性                          |


## 四、xpath的谓语条件（Predicate）

谓语用来查找某个特定的节点或者包含某个指定的值的节点。
谓语被嵌在方括号中。

实例:
* /bookstore/book[1] 选取属于 bookstore 子元素的第一个 book 元素。 
* /bookstore/book[last()] 选取属于 bookstore 子元素的最后一个 book 元素。
* /bookstore/book[last()-1] 选取属于 bookstore 子元素的倒数第二个book元素。
* /bookstore/book[position()<3] 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。
* //title[@lang] 选取所有拥有名为 lang 的属性的 title 元素。
* //title[@lang='eng'] 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。
*  /bookstore/book[price>35.00] 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。
*   /bookstore/book[price>35.00]/title 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。


## 五、选取未知节点(通配符)

    + *   匹配任何元素节点。
    + @*  匹配任何属性节点。
    + node()  匹配任何类型的节点。


实例:

* //book/title | //book/price     选取 book 元素的所有 title 和 price 元素。
* //title | //price               选取文档中的所有 title 和 price 元素。
* /bookstore/book/title | //price 选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。


