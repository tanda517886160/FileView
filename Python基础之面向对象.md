#  Python基础之面向对象

```py
# /usr/bin/python
# -*- coding:utf-8 -*-

class Person():
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender

    def talking(self):
        print("我叫%s，今年%s岁，我是%s生"%(self.name,self.age,self.gender))
```


## 特殊的封装

### 1、classmethod 类方法 

* 方式：在类前使用关键词 `@classmethod`，将方法更改为类方法。
* 与普通类的不同点：类方法只能使用全局下的属性，而不能使用构造方法中的self.属性
* 举例：

```
class Person():
    name="li"
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender
    #若在前加@classmethod,则只能使用全局下的name，而不能使用构造函数中的self.name
    @classmethod
    def talk(self):
        print("%s is talking"%self.name)

if __name__ == '__main__':
    Person.talk()
```

### 2、property 属性方法

* 方式： 在类前面使用关键词 `@property`，将方法改为属性方法
* 与普通方法的不同：在将方法变为一个属性方法后，调用时就不需要加括号了。 直接调用即可。
* 调用时不使用括号这种形式正跟属性调用类似，而且配合setter装饰器，可以有效的控制重要函数值的传参。
举例：

```py
class Person():
    name="li"
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender

    @property
    def talk(self):
        print("%s is talking"%self.name)


#调用如下：
if __name__ == '__main__':
    d = Person("liu",26,"male")
    d.talk
```

结合setter装饰器：

```py
class Person():
    #设定登录名是从认证函数中调取出来的，不允许私自更改。
    @property
    def description(self):
        self.__username="wanglei"
        return self.__username

    @description.setter
    def decs(self,value):
        self.__username=value
        return self.__username

#调用如下：
if __name__ == '__main__':
    d=Person()
    d.description="zhangsan"
    print(d.description)
```


### 3、staticmethod 静态方法

* 方式：在类前面使用关键词 `@staticmethod`，将方法改为静态方法
* 与普通方法的不同：这样写完以后，方法实际上跟类没什么关系了。相当于跟类的关系截断了。只是名义上归类管理，实际上在静态方法里访问不了实例中的属性，只能获取到类中的属性。
* 举例：

```py
class Person():
    #设定登录名是从认证函数中调取出来的，不允许私自更改。
    user="laowang"
    def test(self):
        self.username="laoli"

    @staticmethod
    def statictest(self):
        print(self.username,",Hello")


class Person2():
    #设定登录名是从认证函数中调取出来的，不允许私自更改。
    user="laowang"
    @staticmethod
    def statictest(self):
        print(self.user,",Hello")

#调用如下：
if __name__ == '__main__':
    d=Person()
    d.statictest(d)  #报错， 提示无法找到属性username

    d2=Person2()
    d2.statictest(d) #正确
```


### 4、特殊成员方法

类中除了将方法使用特殊形式封装外，还提供了特殊的成员方法，其形式主要是以为  `__方法名__`  这种形式存在，现在主要是介绍下此类特殊成员方法

####　1）、`__doc__`
* 作用： 打印这个类的描述信息
* 示例：

```py
class Person():
    __doc__ = "存储了人的行为"
    def __init__(self,name):
        self.name=name
    def damajiang(self):
        print("%s在打麻将"%self.name)
    def dushu(self):
        print("%s在读书"%self.name)

if __name__ == '__main__':
    d=Person("zhangsan")
    print(d.__doc__)
```

#### 2）、`__moudile__`

* 作用：表示当前操作的对象在哪个模块中
* 示例：使用之前写过的一个作业来进行说明。首先开始程序是在一个文件夹下，调用的核心程序是在另一个文件夹下的classset.py文件内，使用__module__就可以直接看到调用了哪个模块的内容。

```py
while True:#开启程序的界面
        msg=[]
        print(display)
        select=input("请选择：").strip()
        if select=="1" or select=="管理界面":
            msg.append("administrator")
            msg.append(None)
            adminmgr=AdminMgr(msg)
            print(adminmgr.__module__)
            adminmgr.maincore(msg)
```


#### 4）、`__init__`

* 作用：构造函数，也成为构造方法
**前面已经讲完，不再重复叙述。

 

#### 5）、`__del__` 析构函数或者是析构方法

* 作用：当对象在内存中被释放时，自动触发执行。
* 示例：
可用文字描述下使用场景：比如软件是以server和client的模式存在，当前两者都正常运行，现在server端想要停机维护，如果直接停机的话，可能会引起当前仍旧在线的客户端传输出现问题，那么此时在server在退出时，可以设置析构函数，告诉server在退出时，发送断开信息给客户端，然后再将所有客户端连接断开，这样客户端在收到信息后，可执行自身无法连接至服务器的流程，消除了两者都可能引起崩溃的情况。

#### 6）、`__call__` 

* 作用：对象后面加扩展，直接执行
* 示例：

```py
class Person():
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender
    def talk(self):
        print("%s is talking"%self.name)

    def __call__(self, *args, **kwargs):
        print("this is %s call"%self.name)

d=Person("liu",26,"male")
d()
```

#### 7）、`__dict__`
* 作用：查看对象中所有的成员　　
    `类.__dict__` 打印类里的所有属性，不包括实例属性
    `实例.__dict__` 打印所有实例属性，不包括类里的属性
* 示例:

```py
class Person():
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender
    def talk(self):
        print("talking")

d=Person("liu",26,"male")
print(d.__dict__)
print(Person.__dict__)
```


#### 8）、`__str__`
* 作用：打印对象时，会直接将__str__内的返回值打印出来。
* 示例：

```py
class Person():
    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender
    def talk(self):
        print("talking")
    def __str__(self):
        return "123"

d=Person("liu",26,"male")
print(d)
```

#### 9）、`__getitem__`

* 作用：用于索引操作，表示获取数据
* 示例：

```py
class Value():
    def __init__(self):
        self.dict={
            1:"A",
            2:"B",
            3:"C",
            4:"D"
        }
    def __getitem__(self, item):
        if item in self.dict.keys():
            return self.dict[item]

s=Value()
print(s[1])
```


#### 10）、`__setitem__`
* 作用：用于索引操作，表示设置数据
* 示例：

```py
# -*- coding:utf-8 -*-

class Value():
    def __init__(self):
        self.dict={
            1:"A",
            2:"B",
            3:"C",
            4:"D"
        }

    def valuestr(self):
        return self.dict
    def __setitem__(self, key, value):
        self.dict[key]=value
    def __getitem__(self, item):
        if item in self.dict.keys():
            return self.dict[item]



s=Value()
s[1]="qwe"
print(s[1])
print(s.valuestr())
```

#### 11）、`__delitem__`
* 作用：删除给定键对应的元素。
* 示例：

```py
class Value():
    def __init__(self):
        self.dict={
            1:"A",
            2:"B",
            3:"C",
            4:"D"
        }

    def valuestr(self):
        return self.dict
    def __setitem__(self, key, value):
        self.dict[key]=value
    def __getitem__(self, item):
        if item in self.dict.keys():
            return self.dict[item]
    def __delitem__(self, key):
        del self.dict[key]


s=Value()
s[1]="qwe"
del s[2]
print(s.valuestr())
```

#### 12）、`__iter__`

* 作用：用于迭代器，之所以列表、字典、元组可以进行for循环，是因为类型内部定义了 `__iter__`

#### 13）、`__new__`
* 作用:是在新式类中出现的方法，它作用在构造方法建造实例之前.
    `__new__()` 可以决定是否 要使用当前类`__init__()`方法，因为`__new__()`同样也可以调用其他类的构造方法或者直接返回别的对象来作为本类 的实例。

　　从继承的角度来说，如果类在定义时没有重写`__new__()`方法，Python默认调用该类的父类`__new__()`方法来构造该类实例，若父类中也未进行重写，那么就一直追溯至object的 `__new__()`方法，因为object是所有新式类的基类。

    而如果新式类中重写了`__new__()`方法，那么可以在重写 `__new__()`方法后，调用任意新式类的__new__()方法来进行实例的创造。
 
#### 14）、`__metaclass__`
* 作用:`__metaclass__`称为“元类”，元类就是用来创建类的东西。
* 可以这样简单理解： 
    MyClass = MetaClass()
    MyObject = MyClass()

```py
class Person():

    def __init__(self,name,age,gender):
        self.name=name
        self.age=age
        self.gender=gender
    def talk(self):
        print("talking")


d=Person("liu",26,"male")
print(type(d))
print(type(Person))
#在以上函数中，d通过Person成为实例化对象，在此，d跟Person其实都是一个实例。通过打印d 和Person的类型来看，二者都是通过类来创建的。
```

```
def __init__(self,name,age,gender):
    self.name=name
    self.age=age
    self.gender=gender

def talk(self):
    print(" %s is talking"%self.name)

def eat(self):
    print("%s is eat, he/she is %s"%(self.name,self.age))

Person=type("Person",(object,),{"talk":talk,"__init__":__init__})
d=Person("liu",25,"male")
d.talk()
# 这两者创建的类最终结果都一样，因此也就了解到了，type实际上就是一个元类，type就是Python在背后用来创建所有类的元类。
```