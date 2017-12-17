---
title: Linux典型应用
copyright: true
date: 2017-12-17 19:09:26
tags: [Linux]
categories: Linux典型应用
---

介绍了在Centos7中使用Linux的一些常用操作

<!--more-->
# 安装虚拟机

下载Centos镜像和VirtualBox，之后启动安装

# 安装后设置

## 网络设置

> 安装完执行**ifconfig**是无法执行的，因为安装的镜像是最小安装，因此需要手动进行设置

首先执行ip addr,查看本机的网卡信息，接着执行

```shell
vi /etc/sysconfig/network-scripts/ifcfg-xxx
# xxx 为上面找到的网卡
```

修改其中的ONBOOT修改为YES，之后重启

```shell
service network restart
```

接着安装net-tools

```shell
yum install net-tools
```

之后就可以使用ifconig命令了


## 替换默认源

安装必要软件

```shell
yum install wget vim
```

进行备份

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载对应版本repo文件, 放入/etc/yum.repos.d/

```shell
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
```

运行以下命令生成缓存
```shell
yum clean all
yum makecache
```

# Linux常用命令

## 软件操作命令

* 软件包管理器：yum
* 安装软件：yum install xxx
* 卸载软件: yum remove xxx
* 搜索软件: yum search xxx
* 清理缓存: yum clean packages
* 列出已安装: yum list
* 软件包信息：yum info xxx

## 服务器硬件资源信息

* 内存：free -m
* 硬盘: df -h
* 负载: w/top
* CPU: cat /proc/cpuinfo

## 文件操作命令

### Linux文件目录结构

* 根目录: /
* 家目录: /home
* 临时目录：/tmp
* 配置目录: /etc
* 用户程序目录: /usr

### 文件操作基本命令

* 查看目录下的文件: ls
* 新建文件: touch
* 新建文件夹: mkdir
* 进入目录: cd
* 删除文件: rm
* 删除文件夹: rm -rf
* 复制: cp
* 移动: mv
* 显示路径: pwd

### VIM常用操作

* 移到行首: G
* 移到行尾: gg
* 删除当前行: dd
* 恢复删除：u
* 复制当前行: yy
* 粘贴: p
* 进入插入模式: i
* 退出模式: esc

### 文件搜索、查找、读取

* 从文件尾部开始读取: tail
* 从文件头部度：head
* 读取整个文件：cat
* 分页读取：more
* 可控分页: less
* 搜索关键字: grep
* 查找文件: find
* 统计个数：wc

### 文件解压缩

文件解压缩主要使用的是tar命令

创建一个压缩文件
```shell
tar -cd xxxxx.tar 包含的文件 包含的文件
```

查看压缩文件中的文件列表
```shell
tar tf xxx.tar
```

解压文件
```shell
tar -xf xxx.tar
```

## 系统用户操作命令

* 添加用户: useradd xxx
* 添加用户：adduser xxx
* 删除用户: userdel -r xxx
* 设置密码: passwd xxx

## 防火墙设置

* 安装防火墙: yum install firewalld
* 启动: service firewalld start
* 检查状态: service firewalld status
* 关闭或禁用: service firewalld stop/disable
* 添加一个服务: firewall-cmd --add-service=xxx
* 移除一个服务: firewall-cmd --remove-service=xxx

## 提权和文件下载和上传

### 提权

首先将账号加入到可提权账号列表中
```shell
visudo

# 然后在文件尾部加上
%用户名  ALL(ALL)  ALL
```
然后就能进行提权了
```shell
sudo yum install vim
```

### 文件下载

直接使用wget即可

### 文件上传

```shell
# 将文件上传到服务器地址
scp 文件 用户名@主机地址:文件保存路径

# 将服务器上的文件下载到本地
scp 用户名@主机地址:文件保存路径 本地路径
```

### 在windows中使用xshell进行文件上传和下载

在服务器中安装
```shell
yum install lrzsz
```

上传
```shell
rz
```

下载
```shell
sz 文件名
```

***

# WebServer安装和配置讲解

## Apache的基本操作

### Apache的安装

* 安装：yum install httpd
* 启动: service httpd start
* 停止: service httpd stop

当启动后，由于防火墙的关系无法访问，因此需要店家规则或者关闭防火墙

```shell
添加
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
重新载入
firewall-cmd --reload
```

### Apache的虚拟主机配置

编辑配置文件
```shell
vim /etc/httpd/conf/httpd.conf
```

在其中写上虚拟主机的配置
```shell
<VirtualHost *:80>
        ServerName www.test.com
        DocumentRoot /data/www
        <Directory "/data/www">
                Options Indexes FollowSymLinks
                AllowOverride None
                Require all granted
        </Directory>
</VirtualHost>
```
然后在网站路径上创建网页，并在机器上配置host
如果上述配置完成后无法访问，则需要
```shell
setenforce 0
```

### 配置Apache伪静态

在配置文件中加入
```shell
LoadModule rewrite_module modules/mod_rewrite.so
```

在配置虚拟主机处加入
```shell
<VirtualHost *:80>
        ServerName www.test.com
        DocumentRoot /data/www
        <Directory "/data/www">
                Options Indexes FollowSymLinks
                AllowOverride None
                Require all granted
                <IfModule mod_rewrite.c>
                        RewriteEngine On
                        RewriteRule ^(.*).php$ index.html
                </IfModule>
        </Directory>
</VirtualHost>
```

重启服务器，这样访问类似于http://www.demo.com/2.php也为跳转到首页

## Nginx基本操作

### Nginx的安装

* 安装：yum install nginx
* 启动: service nginx start
* 停止: service nginx stop
* 重载: service nginx reload

在Nginx安装前需要添加yum的资源库
```shell
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

然后执行
```shell
yum install nginx
```

### nginx站点配置

在/etc/nginx/conf.d下面拷贝一份default.conf为demo.conf作为自定义站点的配置文件, 在其中配置

```shell
server {
    listen       80;
    # 可以进行多端口配置,只需要在下面再加一个listen即可
    # listen 9000;
    # 也可以进行多域名，只要在后面再加入即可
    server_name  www.demo.com;
    location / {
        root   /data/www;
        index  index.html index.htm;
    }
}
```

### nginx伪静态配置

nginx中配置伪静态比较简单只需要在配置文件中加入
```shell
server {
    listen       80;
    server_name  www.demo.com;
    location / {
        root   /data/www;
        index  index.html index.htm;
        rewrite ^(.*)\.php$ /index.html;
    }
}

### nginx反向代理

nginx反向代理只要加入proxy_pass即可

```shell

server {
    listen       80;
    server_name  www.demo.com;
    location / {
        root   /data/www;
        index  index.html index.htm;
        proxy_pass http://blog.jiangyixin.top;
    }

```
这样就能自动代理到http://blog.jiangyixin.top了

### nginx负载均衡

```shell
upstream myblog {
   server 151.101.77.147:80 weight=6;
   server 151.101.77.146:80 weight=2;
   # 可以在这里配置多个server来达到负载均衡的效果
   # weight值越高，该服务器被访问的概率越高
}
server {
    listen       80;
    server_name  www.demo.com;
    location / {
        root   /data/www;
        index  index.html index.htm;
        proxy_pass http://myblog;
}
```

***

# 数据库服务

## Mysql安装与连接

### Mysql的基本操作

* 安装: yum install mysql-community-server
* 启动：service mysqld start/restart
* 停止: service mysqld stop

### Mysql安装

Centos默认安装了mariadb，因此需要先进行卸载
```shell
yum removwe mariadb-libs.x86_64
```

下载Mysql源
https://dev.mysql.com/downloads/repo/yum/

安装Mysql源
```shell
yum localinstall mysql57-community-release-el7-11.noarch.rpm
```

安装Mysql
```shell
yum install mysql-community-server
```

查看Mysql密码
```shell
cat /var/log/mysqld.log | grep password
```

重新设置Mysql密码
```shell
# 如果要设置简单密码需要设置以下参数
set global validate_password_policy=0;
set global validate_password_length=1;
# 重设密码
SET PASSWORD = PASSWORD('root')
```

## 远程连接

如果需要远程连接Mysql需要进行一些设置

### 修改连接用户的HOST
```mysql
update user set host='%' where user='root' and host='localhost';

# 修改完成后刷新权限
flush privileges;
```

### 开放3306端口
```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 刷新防火墙配置
firewall-cmd --reload
```

***

# 缓存服务

## Memcached基本操作

* 安装：yum install memcached
* 启动：memcached -d -l -m -p
* 停止: kill pid

## Redis基本操作

* 安装: 源码编译安装
* 启动: redis-server start/restart
* 停止: redis-server stop
* 客户端: redis-client

```shell
wget http://download.redis.io/releases/redis-4.0.6.tar.gz
tar xzf redis-4.0.6.tar.gz
cd redis-4.0.6
yum insatll gcc
make MALLOC=libc
make install
src/redis-server
```

***

# PHP环境配置

## PHP基础环境配置

### PHP安装

配置源
```shell
rpm -Uvh http://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh http://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

安装fpm
```shell
yum install php56w-fpm
```

安装PHP扩展
```shell
yum install php56w.x86_64 php56w-cli.x86_64 php56w-common.x86_64 php56w-gd.x86_64 php56w-mbstring.x86_64 php56w-mcrypt.x86_64 php56w-mysql.x86_64 php56w-pdo.x86_64 php56w-intl.x86_64 php56w-xsl.x86_64
```

启动
```shell
service php-fpm start
```
## Laravel5环境配置

安装Composer
```shell
curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/local/bin/composer
```

安装Laravel安装器
```shell
composer global require "laravel/installer=~1.1"
```

编写nginx配置文件
```shell
server {
    listen       80;
    server_name  laravel.test.com;
    root /data/www/demo_laravel/public;
    index index.php index.html;
    access_log  /var/log/nginx/laravel.log  main;

    location / {
        try_files $uri/ $uri/index.php?$args;
   }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}

```

进入项目根目录运行
```shell
composer install
php artisan key:generate 
```

授予权限
```shell
chmod -R 777 storage/framework/
chomd -R 777 storage/logs/
```

## Yii2环境配置

编写配置文件

```shell
server {
    listen       80;
    server_name  yii.jiangyx.com;
    root /data/www/demo_yii2/web;
    index index.php index.html;
    access_log  /var/log/nginx/yii2.log  main;

    location / {
	try_files $uri/ $uri/index.php?$args;
   } 

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}

```

赋予权限
```shell
chmod -R 777 runtime
chmod -R 777 web/assets/
```

## TP5环境配置

编写配置文件
```shell
server {
    listen       80;
    server_name  tp.jiangyx.com;
    root /data/www/demo_tp/public;
    index index.php index.html;
    access_log  /var/log/nginx/yii2.log  main;

    location / {
        try_files $uri/ $uri/index.php?$args;
   }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}

```

# Java运行环境配置

## JDK安装
```java
sudo yum install java-1.8.0-openjdk*
```

## tomact安装

官网下载解压即可

## maven安装

官网解压即可
```shell
# 做一个软链
ln -s /maven/bin/mvn /usr/bin/mvn
```

***

# Python运行环境

## Python安装

下载的就是源码包
```shell
tar -xvzf Python-3.5.1.tgz
cd Python-3.5.1/
./configure --prefix=/usr/python
yum install zlib zlib-devel
make
make install

替换原有的Python2
mv /usr/bin/python /usr/bin/python.bak
ln -s /usr/python/bin/python3 /usr/bin/python
```

也可以通过一键安装脚本来进行安装
```shell
wget https://raw.githubusercontent.com/LunacyZeus/Python3.6-for-Centos7.0/master/install.sh && sh install.sh
```

## virtualenv安装

```shell
pip install virtualenv

virtualenv blog
```

## pip换成豆瓣源

```shell
mkdir ~/.pip
vim ~/.pip/pip.conf
[global]
timeout=60
index-url=https://pypi.douban.com/simple
```

## 创建虚拟环境和激活

```shell
virtualenv blog

source blog/bin/activate
```

***

#  服务管理

## Crontab定时任务
```shell
*/30 * * * * ntpdate cn.pool.ntp.org
```

## Ntpdate时间校准

```shell
ntpdate cn.pool.ntp.org
```

## Logrotate 日志切分

## Supervisor服务管理
