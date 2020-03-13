---
title: 安装mysql并可外网访问及注意事项
date: 2019-03-03 04:49:41
thumbnail: 
categories:
    - 中间件
tags:
    - mysql
---

## 安装

可以通过yum安装，也可以下载解压版本安装。

解压版本安装 ：

1. 下载：
``` bash 
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz
```

2. 解压， 最好下载的时候就找好目录，避免移动
``` bash 
tar -xvf mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz 
```

3. 创建mysql数据目录（这里注意不要在里面加一些无关的东西）如果不是使用root账号，需要赋予权限
``` bash
cd /
mkdir data
cd data
mkdir mysql
```

4. 配置
``` bash 
vim /etc/my.cnf


[mysqld]
bind-address=0.0.0.0
port=3306
user=root
#安装路径
basedir=/usr/local/mysql-5.7.26
#数据目录
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#编码设置
character_set_server=utf8mb4
```

5. 初始化
``` bash
cd /usr/local/mysql-5.7.26/bin/

./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql-5.7.26/ --datadir=/data/mysql/ --user=root --initialize
```

6. 获取密码(就是配置的mysql数据目录中的err文件)
``` bash
vim /data/mysql/mysql.err
```

找到最后面的密码（如果没有多余操作应该在最后）

7. 启动mysql服务
``` bash
serivce mysqld start
```

这里可能发生的状况：Failed to start mysqld.service: Unit not found
网上的做法是让安装mariadb, 我为什么不直接安装他...这里可以使用：

``` bash
/usr/local/mysql-5.7.26/support-files/mysql.server start/stop 
```

来启动或者关闭服务；暂时还不知道service或者是systemctl的启动为什么会上述错误以及更好的解决方案；

8. 进入mysql
``` bash
mysql -u root -p

# 修改密码
SET PASSWORD = PASSWORD('123456');
# 这里的 % 代表所有域名可访问（其他ip连入），localhost的话只能本机访问
grant all privileges on *.* to 'root'@'%' PASSWORD EXPIRE NEVER;
# 刷新权限
flush privileges;

```
