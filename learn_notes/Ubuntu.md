# 安装ssh
```sh
ubuntu默认不能远程登录
sudo apt-get install openssh-server -y
ps -e |grep ssh

首先防火墙添加允许的端口
sudo ufw allow 7980
sudo vi /etc/ssh/sshd_config
修改端口为7980
sudo service ssh restart
netstat -a | grep 7980

sudo iptables -L
sudo ufw default deny
sudo ufw allow 53
sudo ufw delete allow 53

sudo ufw allow 80/tcp
sudo ufw delete allow 80/tcp

sudo ufw allow from 192.168.254.254
sudo ufw delete allow from 192.168.254.254
```
# 安装gcc
```sh
sudo apt-get install -y build-essential --fix-missing
```
# 设置静态ip
```sh
vi /etc/network/interfaces
这里注意可能不是eth0，找dhcp那个
iface eth0 inet static
address 192.168.1.100    
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameserver 192.168.1.1

sudo /etc/init.d/networking restart
```
# ubuntu1604 vpn server  UDP 1701 500 4500
##### 安装
```sh
apt-get -y install strongswan xl2tpd ppp lsof
echo "net.ipv4.ip_forward = 1" | tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.accept_redirects = 0" | tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.send_redirects = 0" | tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.rp_filter = 0" | tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.accept_source_route = 0" | tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.send_redirects = 0" | tee -a /etc/sysctl.conf
echo "net.ipv4.icmp_ignore_bogus_error_responses = 1" | tee -a /etc/sysctl.conf
for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
```
##### vi /etc/ipsec.conf
```sh
version 2 # conforms to second version of ipsec.conf specification

config setup
conn L2TP-PSK-noNAT
    authby=secret
    #shared secret. Use rsasig for certificates.

    auto=add
    #the ipsec tunnel should be started and routes created when the ipsec daemon itself starts.

    keyingtries=3
    #Only negotiate a conn. 3 times.

    ikelifetime=8h
    keylife=1h

    ike=aes256-sha1,aes128-sha1,3des-sha1

    type=transport
    #because we use l2tp as tunnel protocol

    left=%any
    #fill in server IP above

    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any

    dpddelay=10
    # Dead Peer Dectection (RFC 3706) keepalives delay
    dpdtimeout=20
    #  length of time (in seconds) we will idle without hearing either an R_U_THERE poll from our peer, or an R_U_THERE_ACK reply.
    dpdaction=clear
    # When a DPD enabled peer is declared dead, what action should be taken. clear means the eroute and SA with both be cleared.
```

##### vi /etc/ipsec.secrets
```sh
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

%any : PSK "PASSWORD"
```
##### vi /etc/xl2tpd/xl2tpd.conf
```sh
[global]
ipsec saref = yes
saref refinfo = 30

;debug avp = yes
;debug network = yes
;debug state = yes
;debug tunnel = yes

[lns default]
ip range = 192.168.100.100 - 192.168.100.200
local ip = 192.168.100.1
refuse pap = yes
require authentication = yes
;ppp debug = yes
pppoptfile = /etc/ppp/options
length bit = yes
```
##### vi /etc/ppp/options
```sh
require-mschap-v2
ms-dns 114.114.114.114
auth
mtu 1200
mru 1000
crtscts
hide-password
modem
name l2tpd
proxyarp
lcp-echo-interval 30
lcp-echo-failure 4
```

##### vi /etc/ppp/chap-secrets
```sh
# Secrets for authentication using CHAP
# client server secret IP addresses
zhangsan l2tpd password1 *
```
##### 启动
```sh
ipsec restart
service xl2tpd restart
ipsec statusall
/var/log/syslog
/var/log/auth.log

```
##### win10连接
```sh
无法建立计算机与 VPN 服务器之间的网络连接，因为远程服务器未响应。这可能是因为未将计算机与远程服务器之间的某种网络设备(如防火墙、NAT、路由器等)配置为允许 VPN 连接。请与管理员或服务提供商联系以确定哪种设备可能产生此问题。

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent  dword AssumeUDPEncapsulationContextOnSendRule=2

HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters  dword ProhibitIpSec=1
去掉在远程网络上使用默认网关
```
# 虚拟机共享
```sh
sudo apt-get install open-vm-dkms
```
# 改源，用阿里云的最后好像有几个会报错，只编辑一下/etc/apt/sources.list添加清华和中科大的即可
```sh
http://mirrors.aliyun.com/repo/
sudo apt-get install -y wget
sudo mv /etc/apt/sources.list /etc/apt/sources-bak.list
sudo wget -O /etc/apt/sources.list http://mirrors.aliyun.com/repo/ubuntu1404-lts.list
添加清华和中科大的
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates main restricted
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates main restricted
deb http://mirrors.ustc.edu.cn/ubuntu/ precise universe
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise universe
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates universe
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates universe
deb http://mirrors.ustc.edu.cn/ubuntu/ precise multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu precise-security main restricted
deb-src http://security.ubuntu.com/ubuntu precise-security main restricted
deb http://security.ubuntu.com/ubuntu precise-security universe
deb-src http://security.ubuntu.com/ubuntu precise-security universe
deb http://security.ubuntu.com/ubuntu precise-security multiverse
deb-src http://security.ubuntu.com/ubuntu precise-security multiverse
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse

```
# 阿里云安装docker，感谢阿里云，一行命令搞定
```sh
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
提示If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:
sudo usermod -aG docker zhangtao
如果提示有lock无法访问就删掉对应的lock文件
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://0rnhdnox.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo apt-get update
```
# 官方安装docker，流程复杂，慢到你想哭
```sh
sudo apt-get update
sudo apt-get -y install \
  apt-transport-https \
  ca-certificates \
  curl

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
如果上一条命令很慢就分成下面两步
wget https://download.docker.com/linux/ubuntu/gpg --no-check-certificate 
sudo apt-key add gpg
如果还是很慢就多执行几次

sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
sudo apt-get update
如果执行很慢或者失败就多执行几次
sudo apt-get -y install docker-ce

可能是过时的命令
sudo apt-get update && sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
sudo apt-get install -y docker-engine

wget -qO- https://get.docker.com/ | sh

curl https://raw.githubusercontent.com/docker/docker/master/hack/install.sh | bash
```

# 阿里云ubuntu服务器
```sh
apt-get install vsftpd
whereis vsftpd

vi /etc/vsftpd.conf
write_enable=YES

vi /etc/ftpusers  这个文件是禁止连接的用户表
#root

vi /var/log/vsftpd.log
可查看是否成功登录
如果显示无法显示远程文件夹，把会话属性的使用被动模式选项去掉
```

# ubuntu1604 tomcat8 nginx
```sh
apt-cache search tomcat
atp-get -y install tomcat8
安装目录为/usr/share/tomcat8
cp -r app1name /var/lib/tomcat8/webapps/
/var/lib/tomcat8/webapps/app1name
chmod -R +rx app1name

/etc/tomcat8/server.xml
/etc/hosts

apt-cache search nginx
apt-get -y install nginx
/etc/nginx/nginx.conf

systemctl start tomcat8
实时查看tomcat输出
tail -f /var/log/tomcat8/catalina.out
systemctl start nginx

netstat -na | grep ':80.*LISTEN'
nginx -t
nginx -s reload

 protocol="org.apache.coyote.http11.Http11NioProtocol"

```
# dockernginx
```sh
/etc/nginx
/var/log/nginx
```
# dockertomcat
```sh
/usr/local/tomcat/webapps
```
# SSL
```sh

nginx
 server {
    listen 443;
    server_name www.web.com;
    rewrite ^(.*)$  https://$host$1 permanent;
    ssl on;
    ssl_certificate   /etc/nginx/cert/123.pem;
    ssl_certificate_key  /etc/nginx/cert/123.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
         proxy_pass http://app1name:8080;
    }
}


tomcat  不用安装JKS，页面中如果调用了其他网站资源也可能会提示不是安全链接
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
                 SSLEnabled="true"
                 scheme="https"
                 secure="true"
                keystoreFile="/usr/share/tomcat8/cert/123.pfx"
                keystoreType="PKCS12"
                 keystorePass="keypass"    
                 clientAuth="false"
    SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
ciphers="TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
SSLCipherSuite="ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4"

```
# 运行多个jar包
```sh
nohup java -jar 启动1.jar &
nohup java -jar 启动2.jar &
nohup java -jar 启动3.jar &
```