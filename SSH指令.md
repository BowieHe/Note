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

