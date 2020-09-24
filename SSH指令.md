## extplugin插件测试环境Redis隧道

ssh -L 6378:192.168.31.209:6379 root@jenkins.plugin.xsio.cn -p 10022

## impala测试环境隧道

ssh -L 21050:hadoop02:21050 dmtb

ssh hadoop02

impala-shell

## 堡垒机

Host  dmtb
    HostName 114.55.125.150
    User  hebowei
    port 10022
    ProxyCommand    none
    IdentityFile   ~/.ssh/id_rsa
    PasswordAuthentication    no

vim  ~/.ssh/config

地址：rm-bp19r2t20xzz0b24z.mysql.rds.aliyuncs.com
帐号：hebowei
密码：irTs2anGKe5HCyum

ssh host： 114.55.124.253
user： hebowei
ssh password：选到自己的私钥
port： 10022



地址：rm-bp1401ebe70rax76n.mysql.rds.aliyuncs.com
帐号：hebowei
密码：irTs2anGKe5HCyum
ssh host： 116.62.218.80
user： hebowei
ssh password：选到自己的私钥
port： 10022



r-bp132f183e4eac34.redis.rds.aliyuncs.com
Xiaoshu406



ssh -L 6379:r-bp132f183e4eac34.redis.rds.aliyuncs.com:6379 hebowei@jenkins.xsio.cn -p 10022



redis_host: 192.168.31.209
 redis_password: Xiaoshu406
 ssh -L 6379:192.168.31.209:6379 root@jenkins.plugin.xsio.cn -p 10022



Your Server IP        :  23.100.100.34
Your Server Port      :  443
Your Password         :  CkGA3TFTiz59wZZ
Your Encryption Method:  aes-256-cfb



sre.dmhub.cn
bowei.he@convertlab.com
ojpYo4iJy2E5NvFa



地址： rm-bp1708x44rq0bdapn.mysql.rds.aliyuncs.com
帐号： hebowei
密码： SUEXpsyZJM3ZN7Qa



Kafka,redis,zookeeper,mysql, mongoldb

