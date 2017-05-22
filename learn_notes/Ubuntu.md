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