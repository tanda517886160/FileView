# python环境之pip安装

### PIP简介

pip 是"A tool for installing and managing Python packages"，也就是说pip是python的软件安装工具


### 下载pip
```
# cd /usr/local/src
# wget "https://pypi.python.org/packages/source/p/pip/pip-1.5.4.tar.gz#md5=834b2904f92d46aaa333267fb1c922bb" --no-check-certificate

# tar -xzvf pip-1.5.4.tar.gz
# cd pip-1.5.4
# python setup.py install
```

> 如果安装报下面的错,那么就要先安装setuptools包：
```py
Traceback (most recent call last):
  File "setup.py", line 6, in <module>
    from setuptools import setup, find_packages
ImportError: No module named setuptools
```

下载setuptools包，并安装
```
# wget http://pypi.python.org/packages/source/s/setuptools/setuptools-2.0.tar.gz

# //解压setuptools包
# tar zxvf setuptools-2.0.tar.gz
# cd setuptools-2.0

# //编译setuptools
# python setup.py build

# //开始执行setuptools安装
#  python setup.py install

# //重新执行
#  cd /usr/local/src/pip-1.5.4
#  python setup.py install
```



Windows 安装pip
```
#下载pip源码：
http://pypi.python.org/pypi/pip/

#或者直接下载
http://pypi.python.org/packages/source/p/pip/pip-6.0.8.tar.gz#md5=2332e6f97e75ded3bddde0ced01dbda3

下载下来后解压到指定目录，打开命令行：
cd D:\Python27\pip-6.0.8

#编译安装
python setup.py install 

#安装python包
python -m pip install <package name>
```


