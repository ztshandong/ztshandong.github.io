
# 关闭SELINUX
```sh
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出 
setenforce 0 #使配置立即生效
```
# vim
```sh
?string 向下查找
/string 向上查找
n 同向搜索  N逆向搜索
```
# sudo
```redhat
默认使用sudo会出现：username不在 sudoers 文件中。此事将被报告
步骤一：
su root
步骤二：
方法一：（推荐）
visudo
方法二：
su root
chmod u+w /etc/sudoers 改完后要chmod u-w /etc/sudoers
vi /etc/sudoers
步骤三：
找到root ALL=(ALL) ALL后在下面添加
username ALL=(ALL) ALL
```
# network
```centos
centos网卡默认不启用
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 (网卡名可能不同)
ONBOOT=yes
然后重启网卡
systemctl restart network
```
# yum
```unix
yum的命令形式一般是如下：yum [options] [command] [package ...]
其中的[options]是可选的，选项包括-h（帮助），-y（当安装过程提示选择全部为"yes"），-q（不显示安装的过程）等等。[command]为所要进行的操作，[package ...]是操作的对象。

概括了部分常用的命令包括：

自动搜索最快镜像插件：   yum install yum-fastestmirror
安装yum图形窗口插件：    yum install yumex
查看可能批量安装的列表： yum grouplist

1 安装
yum install 全部安装
yum install package1 安装指定的安装包package1
yum groupinsall group1 安装程序组group1

2 更新和升级
yum update 全部更新
yum update package1 更新指定程序包package1
yum check-update 检查可更新的程序
yum upgrade package1 升级指定程序包package1
yum groupupdate group1 升级程序组group1

3 查找和显示
yum info package1 显示安装包信息package1
yum list 显示所有已经安装和可以安装的程序包
yum list package1 显示指定程序包安装情况package1
yum groupinfo group1 显示程序组group1信息yum search string 根据关键字string查找安装包

4 删除程序
yum remove &#124; erase package1 删除程序包package1
yum groupremove group1 删除程序组group1
yum deplist package1 查看程序package1依赖情况

5 清除缓存
yum clean packages 清除缓存目录下的软件包
yum clean headers 清除缓存目录下的 headers
yum clean oldheaders 清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) 清除缓存目录下的软件包及旧的headers

```
# 安装modejs
```sh
方法一：
yum -y install gcc gcc-c++ kernel-devel
查看最新版本
https://nodejs.org/zh-cn/download/current/
wget https://nodejs.org/dist/v7.7.3/node-v7.7.3.tar.gz
tar -xf node-v4.5.0.tar.gz
rm -f node-v4.5.0.tar.gz
cd node-v4.5.0
./configure
make
sudo make install
方法二：
su root
curl -sL https://rpm.nodesource.com/setup | bash -
yum install -y nodejs  这样nodejs版本是0.x
方法三：
NVM（Node version manager）顾名思义，就是Node.js的版本管理软件，可以轻松的在Node.js各个版本间切换，
项目源码[GitHub](https://github.com/creationix/nvm)貌似只剩iojs了
先去github上查看最新版本
source ~/.bash_profile
nvm install node 安装最新版
```
# ftp
```sh
rpm -q vsftpd 查看是否安装
yum -y install vsftpd 如果没安装就安装
whereis vsftpd 查看安装路径
systemctl start vsftpd.service
vi /etc/vsftpd/vsftpd.conf
anonymous_enable=NO 默认是YES
local_enable=YES
write_enable=YES
chroot_local_user=YES

systemctl restart vsftpd
systemctl enable vsftpd 开机启动
```
# firewall
```sh
使用自带防火墙：CentOS7使用的是Linux Kernel 3.10.0的内核版本，新版的Kernel内核已经有了防火墙netfilter
方法一
cp /usr/lib/firewalld/services/http.xml /etc/firewalld/services/
firewall-cmd --reload
方法二 间接修改/etc/firewalld/zones/public.xml文件
##Add
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --add-port=21/tcp
firewall-cmd --add-service=ftp
##Remove
firewall-cmd --permanent --zone=public --remove-port=80/tcp
##Reload
firewall-cmd --reload
查看防火墙状态
systemctl status firewalld.service
启动防火墙
systemctl start firewalld.service
关闭防火墙
systemctl stop firewalld.service
firewall-cmd --list-all 
firewall-cmd --add-service=ftp


使用iptables防火墙，要禁用自带防火墙
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
sudo yum install iptables-services
vi /etc/sysconfig/iptables
-A INPUT -m state –state NEW -m tcp -p tcp –dport 80 -j ACCEPT 
systemctl restart iptables.service　　
systemctl enable iptables.service　


```
