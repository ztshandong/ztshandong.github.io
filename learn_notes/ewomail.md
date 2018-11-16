# centos
```
https://dns.console.aliyun.com/#/dns/setting/bugongsoft.com
drjaybig
Bugong123

command routing

https://gitee.com/laowu5/EwoMail
/etc/hosts
127.0.0.1 bugongsoft.net mail.bugongsoft.net smtp.bugongsoft.net imap.bugongsoft.net mailoraps.bugongsoft.net bugongmail bugongmailadmin

vi /etc/sysconfig/selinux
SELINUX=enforcing 改为 SELINUX=disabled
查看swap
[root@mail ~]# free -m
如果swap位置都显示是0，那么系统还没创建swap
创建1G的swap，可以根据你的服务器配置来调整大小
[root@mail ~]#sudo dd if=/dev/zero of=/mnt/swap bs=1M count=1024  
设置交换分区文件
[root@mail ~]#sudo mkswap /mnt/swap
启动swap
[root@mail ~]#sudo swapon /mnt/swap
设置开机时自启用 swap 分区
需要修改文件 /etc/fstab 中的 swap 行，添加
/mnt/swap swap swap defaults 0 0

sudo wget -c http://download.ewomail.com:8282/ewomail-1.05.sh &&sudo sh ewomail-1.05.sh bugongmail.com

ubuntu 不行
./start.sh: 13: set: Illegal option -o pipefail
ls -al /bin/sh
sudo dpkg-reconfigure dash
或者
sudo ln -fs /bin/bash /bin/sh

安装过程中可能会显示 shutting down postfix : FAILED，如果它的下面再出现一条 starting postfix : OK ，那就是正常的。

安装成功后将会输出"Complete installation"。

查看安装的域名和数据库密码
cat /ewomail/config.ini
邮箱管理后台：http://IP:8010 （默认账号admin，密码123456） 
phpadmin http://IP:8020 
web邮件系统：http://IP:8000

/etc/nginx/nginx.conf
worker_processes auto;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.

events {
    worker_connections 1024;
}

http {

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    sendfile            on;
    keepalive_timeout   65;
    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.

server {
        listen       80;
#        listen       80 default_server;
#        listen       [::]:80 default_server;
        server_name  mail.bugongsoft.net;

        location / {
            proxy_pass http://bugongmail:8000;
        }
    }

     server {
        listen       80;
        server_name  mailoraps.bugongsoft.net;

        location / {
            proxy_pass http://bugongmailadmin:8010;
        }
    }

/ewomail/www/ewomail-admin/core/config.php
url是管理网址
webmail_url是邮箱网址
```