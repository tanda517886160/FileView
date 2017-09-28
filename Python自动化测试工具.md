# Python 自动化测试工具


## WebDriver环境搭建
WebDriver是主流Web应用自动化测试框架，具有清晰面向对象 API，能以最佳的方式与浏览器进行交互。
驱动程序，用以启动各浏览器，具体的驱动程序需要对应的驱动，在浏览器官网上可以找到下载地址。

* [npm提供的下载链接](https://www.npmjs.com/package/selenium-webdriver)
* [selenium官网提供的下载链接](http://www.seleniumhq.org/download/)



## Selenium 安装
selenium 是一个web的自动化测试工具。用代码的方式去模拟浏览器操作过程
直接利用pip工具安装：
```py
pip install selenium
```


## Selenium 使用

### 1. 一个基础实例:
```py
#! /usr/bin/python
# -*- coding:utf8 -*-

# 启动浏览器需要用到
from selenium import webdriver
#引入ActionChains鼠标操作类
from selenium.webdriver.common.action_chains import ActionChains
#引入keys类操作(提供键盘按键支持,最后一个K要大写)
from selenium.webdriver.common.keys import Keys 
#引入时间和系统环境
import time
import os

#驱动设置，指定驱动路劲
iepath = os.path.abspath('D:\WebDriver\IEDriverServer.exe')
browser = webdriver.Ie(iepath)

#打开页面
browser.get('http://www.baidu.com')
browser.maximize_window()  # 浏览器最大化

#获取内容
text = browser.find_element_by_id('s_mod_weather').text
print text 
time.sleep(3)

#模拟输入与点击提交
browser.find_element_by_id('kw').send_keys(u'Python')
browser.find_element_by_id('su').click()
time.sleep(3)

#退出
browser.quit()
```

### 2.  查找元素

```py
#通过ID查找
element = browser.find_element_by_id("passwd-id")
#通过标签中的name属性查找
element = browser.find_element_by_name("passwd")
#xpath查找
element = browser.find_element_by_xpath("//input[@id='passwd-id']")
#通过链接文本获取超链接
#如：<a href="continue.html">Continue</a>
element = browser.find_element_by_link_text("Continue")
element = browser.find_element_by_partial_link_text("Conti")
#通过标签名查找元素
element = browser.find_element_by_tag_name()
#通过Class name 定位元素
element = browser.find_element_by_class_name()
#通过CSS选择器查找元素
#如：<p class="content">Site content goes here.</p>
element = browser.find_element_by_css_selector("p.content")
```

> [W3C XPath Recommendation](https://www.w3.org/TR/xpath/)  
> [XPath Tutorial](http://www.zvon.org/comp/r/tut-XPath_1.html/)- with interactive examples.  
> [XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl)- for Google Chrome   
> [CSS选择器介绍](http://saucelabs.com/resources/articles/selenium-tips-css-selectors)  

查找多个，数组形式:

```py
# 一次查找多个元素 (这些方法会返回一个list列表):
elements = browser.find_elements_by_name()
elements = browser.find_elements_by_xpath()
elements = browser.find_elements_by_link_text()
elements = browser.elements = browser.find_elements_by_partial_link_text()
elements = browser.find_elements_by_tag_name()
elements = browser.find_elements_by_class_name()
elements = browser.find_elements_by_css_selector()
```

除了上述的公共方法，下面还有两个私有方法：find_element 和 find_elements

```py
from selenium.webdriver.common.by import By
driver.find_element(By.XPATH, '//button[text()="Some text"]')
driver.find_elements(By.XPATH, '//button')

#By 类的一些可用属性:
ID = "id"
XPATH = "xpath"
LINK_TEXT = "link text"
PARTIAL_LINK_TEXT = "partial link text"
NAME = "name"
TAG_NAME = "tag name"
CLASS_NAME = "class name"
CSS_SELECTOR = "css selector"
```



> ** 当你使用`XPATH`时，你必须注意，如果匹配超过一个元素，只返回第一个元素。 如果上面也没找到，将会抛出 ``NoSuchElementException`异常。**

### 3. 页面交互

* 简单交互：

```py
#键盘输入
element.send_keys("some text")
#通过”Keys”类来模式输入方向键
element.send_keys("and some key", Keys.ARROW_DOWN)
#清除输入框中的内容 (input或者textarea元素)
element.clear()

```

* 下拉框选择：

```py
element = browser.find_element_by_xpath("//select[@name='name']")
all_options = element.find_elements_by_tag_name("option")
for option in all_options:
    print("Value is: %s" % option.get_attribute("value"))
    option.click()

#或者
from selenium.webdriver.support.ui import Select
select = Select(browser.find_element_by_name('name'))
select.select_by_index(index)
select.select_by_visible_text("text")
select.select_by_value(value)

#取消选择已经选择的元素
select = Select(browser.find_element_by_name('name'))
select.deselect_all() #这将取消选择所以的OPTION

#列出所有已经选择的选项
select = Select(browser.find_element_by_xpath("//select[@name='name']"))
all_selected_options = select.all_selected_options

#获得所以选项,包括未选中的选项
options = select.options

#提交表单
browser.find_element_by_id("submit").click()
#WebDriver对每一个元素都有一个叫做 “submit” 的方法
#或者如果你在一个表单内的元素上使用该方法，WebDriver会在DOM树上就近找到最近的表单并提交它
# 如果调用的元素不再表单内，将会抛出``NoSuchElementException``异常:
element.submit()
```


* 拖拽效果：

```py
from selenium.webdriver import ActionChains
action_chains = ActionChains(browser)

# 移动一个元素
element = browser.find_element_by_id("source")
target = browser.find_element_by_id("target")
# 拖动到指定的目标节点，并渲染
action_chains.drag_and_drop(element, target).perform()
```

* 窗体移动:

```py
#WebDriver 支持在不同的窗口之间移动，只需要调用``switch_to_window``方法即可:
browser.switch_to_window("windowName")
#所有的 driver 对象将会指向当前窗口，可以迭代所有已经打开的窗口
for handle in browser.window_handles:
    browser.switch_to_window(handle)
#可以在不同的frame中切换
browser.switch_to_frame("frameName")
#通过“.”操作符你还可以获得子frame，并通过下标指定任意frame
browser.switch_to_frame("frameName.0.child")
#返回父frame
browser.switch_to_default_content()

#弹出对话框
alert = browser.switch_to_alert()
```

* 浏览器操作:

```py
#访问浏览器历史记录

#前进
browser.forward()
#后退
browser.back()
```

Cookies:

```py
#设置Cookies
cookie = {"name" : "foo", "value" : "bar"} 
browser.add_cookie(cookie)
#获取所有当前URL下可获得的Cookies
browser.get_cookies()
```


### 3. 等待页面加载
Selenium Webdriver 提供两种类型的waits - 隐式和显式。显式等待会让WebDriver等待满足一定的条件以后再进一步的执行。而隐式等待让Webdriver等待一定的时间后再才是查找某元素。

* 显式等待:

显式等待是你在代码中定义等待一定条件发生后再进一步执行你的代码。
如：`time.sleep()` 将条件设置为等待一个确切的时间段
更方便的方法让你只等待需要的时间。WebDriverWait结合ExpectedCondition 是实现的一种方式。

```py
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("http://somedomain/url_that_delays_loading")
try:
    element = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "myDynamicElement"))
    )
finally:
    driver.quit()
```

在抛出TimeoutException异常之前将等待10秒或者在10秒内发现了查找的元素。 WebDriverWait 默认情况下会每500毫秒调用一次ExpectedCondition直到结果成功返回。 ExpectedCondition成功的返回结果是一个布尔类型的true或是不为null的返回值。
expected_conditions 模块提供了一组预定义的条件供WebDriverWait使用。


* 隐式等待:

如果某些元素不是立即可用的，隐式等待是告诉WebDriver去等待一定的时间后去查找元素。 默认等待时间是0秒，一旦设置该值，隐式等待是设置该WebDriver的实例的生命周期。

```py
from selenium import webdriver

driver = webdriver.Chrome()
driver.implicitly_wait(10) # seconds
driver.get("http://somedomain/url_that_delays_loading")
myDynamicElement = driver.find_element_by_id("myDynamicElement")
```
