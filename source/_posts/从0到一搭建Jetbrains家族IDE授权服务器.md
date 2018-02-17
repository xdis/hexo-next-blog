---
title: 从0到一搭建Jetbrains家族IDE授权服务器
copyright: true
date: 2017-12-17 19:09:26
tags: [破解]
categories: 黑科技
---

介绍了如何搭建属于自己的激活服务器

<!--more-->

# 下载破解文件

```
下载地址：

https://mega.nz/#!f4A2WQRB!fMNbcuSt0YxrjXclW81_GZol-g6dURrO1htqXPMYa8Q

链接: https://pan.baidu.com/s/1dFS9DaL 密码: xxc2
```

# 开始运行

下载后有很多版本，如果你电脑是windows，对应的使用windows后缀的，Mac OS使用darwin后缀，

os x 10.12上需要把upx加的壳脱掉，用高点的端口

``` shell
brew install upx
px -d IntelliJIDEALicenseServer_darwin_amd64
```

Ubuntu/centos等没有对应后缀的用linux，要注意区别32/64位,amd64是64位，386是32位。

windows下就不介绍了，点击就可以用，如果需要自定义参数，请根据采用命令行带参数运行,，参数如下：

```
-l 指定绑定监听到哪个IP(私人用)
-u 用户名参数，当未设置-u参数，且计算机用户名为^[a-zA-Z0-9]+$时，使用计算机用户名作为idea用户名
-p 参数，用于指定监听的端口
-prolongationPeriod 指定过期时间参数
```

> 若在程序工作目录中存在IntelliJIDEALicenseServer.html文件，则返回IntelliJIDEALicenseServer.html中的内容到用户浏览器。

接下来，介绍如何部署到Linux服务器上，首先将IntelliJIDEALicenseServer_linux_amd64上传到任意目录，我这里是root目录，先将名字改了，太长了

``` shell
mv IntelliJIDEALicenseServer_linux_amd64 IdeaServer
chmod +x IdeaServer
```

使用nohub进行后台运行

``` shell
nohup ./IdeaServer -p 1024 -prolongationPeriod 999999999999 >> idea.out 2>&1 &;
```

# 使用nginx进行代理

## 安装nginx

### centos6.x

在/etc/yum.repos.d/目录下创建一个源配置文件nginx.repo

``` shell
cd /etc/yum.repos.d/

vim nginx.repo
```

填写如下内容

``` text
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch/
gpgcheck=0
enabled=1
```
保存，则会产生一个/etc/yum.repos.d/nginx.repo文件。

下面直接执行如下指令即可自动安装好Nginx：

``` shell
yum install nginx -y
```

安装完成，下面直接就可以启动Nginx了：

``` shell
service nginx start
```

现在Nginx已经启动了，直接访问服务器就能看到Nginx欢迎页面了的。

如果还无法访问，则需配置一下Linux防火墙。

``` shell
iptables -I INPUT 5 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

service iptables save

service iptables restart
```

Nginx的命令以及配置文件位置：

``` shell
service nginx start # 启动Nginx服务
service nginx stop # 停止Nginx服务
service nginx.conf # Nginx配置文件位置
```

至此，Nginx已经全部配置安装完成。

### centos7.x

使用yum安装nginx需要包括Nginx的库，安装Nginx的库

``` shell
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

使用下面命令安装nginx

``` shell
#yum install nginx
```

启动Nginx

``` shell
service nginx start

# 或者使用
systemctl start nginx.service
```

## 使用nginx配置反向代理

``` text
{
    listen 80;
    server_name 访问的域名;
 
    location / {
        proxy_pass http://127.0.0.1:1024;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

重启nginx即可