## linux上编译安装python-2.7.13

### 2.7版本安装过程
```
#1. 下载python2.7
wget https://www.python.org/ftp/python/2.7.13/Python-2.7.13.tgz

#2. 解压文件
tar zxvf Python-2.7.1.tgz 

#3. 创建应用文件夹
mkdir /usr/local/python-2.7.1

# 编译
cd Python-2.7.1.tgz 
./configure --prefix=/usr/local/python-2.7.1
make && make install

# 软链接
ln -s /usr/local/python-2.7.1/bin/python  /usr/bin/python
ln -s /usr/local/python-2.7.1  /usr/local/python
```


### 3.6版本安装过程
```
#1. 下载python3.6
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tgz

#2. 解压文件
tar zxvf Python-3.6.2.tgz 

#3. 创建应用文件夹
mkdir /usr/local/python-3.6.2

# 编译
cd Python-3.6.2.tgz 
./configure --prefix=/usr/local/python-3.6.2
make && make install

# 软链接
ln -s /usr/local/python-3.6.2/bin/python3  /usr/bin/python3
ln -s /usr/local/python-3.6.2/bin/pip3  /usr/bin/pip3
```

说明：python3自带了pip3工具，python2 需要自己下载编译pip
