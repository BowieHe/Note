### netstat

```shell
-a  显示所有连接，包括TCP，UDP，和unix协议
-t	只列出TCP协议连接
-u	选项列出UDP协议连接
n		禁用反向域名解析，加快查询速度
-l	只列出正在监听的连接，不能和 -a 同时使用
-p	可以查看进程信息（此时netstat应在root权限下）
-pe	同时查看进程名和进程所属用户名
-r	打印内核路由信息，和route命令输出一样
-i	输出网络接口设备统计信息，类似ifconfig
-s	输出针对不同网络协议的统计信息，含Ip，Icmp，Tcp，Udp

netstat -anp ｜ grep apache2 #查看指定服务是否正常运行
netstat -tulnp｜ grep 330 # MySQL中查看正在监听的多管理
```

### chown

`chown -R mysql.mysql /data/*`授权