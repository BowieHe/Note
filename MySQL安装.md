# 二进制安装

1. 在CentOS上新建目录，存放上传的MySQl软件
   同时Mac上传输文件到CentOS

```shell
mkdir -p /app/
scp /Users/bowei/Documents/Image root@192.168.191.32:~
#传输Image到用root用户登陆192.168.191.32 根目录上，默认ssh的22端口
```

2. 解压并把文件存放在mysql中

```shell
mv mysql-5.7.20-linux-glibc2.12-x86_64 mysql
#mv是move命令，移动文件到mysql文件中，新建mysql文件夹
```

3. 修改环境变量

```shell
vim /etc/profile
export PATH=/app/mysql/bin:$PATH
[root@db01 bin]# source /etc/profile
```

4. 建立mysql用户和组，并创建相关目录，修改权限

```shell
 useradd mysql #创建用户和组
 mkdir /data/mysql -p 
 chown -R mysql.mysql /app/*
 chown -R mysql.mysql /data/*
```

5. 清空目录并初始化数据库

```shell
\rm -rf  /data/mysql/* #清空目录
mysqld --initialize-insecure  --user=mysql --basedir=/app/mysql --datadir=/data/mysql
#初始化数据，初始化管理员密码为空
```

6. 配置默认配置文件;使用systemd管理mysql

```shell
vi /etc/my.cnf  # 配置文件
[mysqld]
user=mysql
basedir=/app/mysql
datadir=/data/mysql
server_id=6
port=3306
socket=/tmp/mysql.sock
[mysql]
socket=/tmp/mysql.sock
prompt=3306 [\\d]>

vim /etc/systemd/system/mysqld.service #使用systemd管理mysql
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000

systemctl  start/stop/restart/status   mysqld #mysql启动关闭方式
```

7. 或者配置启动脚本

```shell
cd /app/mysql/support-files
./mysql.server start
## 打开前需要把其他方式打开的mysql关闭
```

8. 管理员密码

```shell
mysqladmin -uroot -p password xx
```

