# 安装
```sh
建议安装英文
Add-Ons中选择Development Tools即可，其他不用选
选择硬盘界面虽然默认已选，但最好是先取消再重新选择
```
# network
```centos
centos网卡默认不启用
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 (网卡名可能不同)
ONBOOT=yes
然后重启网卡
systemctl restart network
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
记得防火墙添加端口
```
#### 安装Vmtools
```sh
若是未安装图形界面貌似不用装这个
cp vmtools.gz /tmp
cd /tmp
tar -zxf vmtools.gz
cd vm-tools
su root
./vmware-install.pl -d
```
#### 关闭SELINUX
```sh
是否要关闭自己决定
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出 
setenforce 0 #使配置立即生效
```
# 修改SSH端口
```sh
rpm -qa|grep ssh 查看已安装的ssh包
systemctl status sshd.service 查看ssh服务状态
ps -ef |grep sshd 查看ssh进程状态
netstat -anpl |grep sshd 查看端口 -p 参数必须为root
vi /etc/ssh/sshd_config
Port 123
firewall-cmd --permanent --zone=public --add-port=123/tcp 

SELinux 默认只允许 22 端口，使用semanage，来修改 ssh 可访问的端口
sestatus -v |grep SELinux 查看SELinux是否启用
rpm -qa |grep policycoreutils-python 检查是否安装semanage
yum install policycoreutils-python 未安装就安装
semanage port -l |grep ssh 查看允许的ssh端口
semanage port -a -t ssh_port_t -p tcp 123 添加新端口
semanage port -m -t ssh_port_t -p tcp 444 修改为其他端口
semanage port -l |grep ssh 查看是否添加成功

firewall-cmd --reload
firewall-cmd --zone=public --list-all
systemctl restart firewalld.service 重启防火墙
systemctl restart sshd.service 重启ssh服务

使用新端口登录后删除22
firewall-cmd --permanent --zone=public --remove-port=22/tcp 
semanage port -d -t ssh_port_t -p tcp 22 报错，semange 不能禁用 ssh 的 22 端口
firewall-cmd --reload
firewall-cmd --zone=public --list-all
systemctl restart firewalld.service 重启防火墙
```
# vim
```sh
i进入编辑模式
ESC进入查看模式
查看模式中:进入命令模式
:q!直接退出
:wq保存退出
?string 向下查找
/string 向上查找
n 同向搜索  N逆向搜索
```
# sudo
```sh
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
# rpm
```sh
rpm包，由“-”、“.”构成，包名、版本信息、版本号、运行平台
 
对已安装软件信息的查询
rpm -qa                             查询已安装的软件
rpm -qf 文件名绝对路径              文件名的绝对路径
rpm -ql 软件名                        查询已安装的软件包都安装到何处
rpm -qi 软件名                        查询一个已安装软件包的信息
rpm -qc 软件名                        查看已安装软件的配置文件
rpm -qd 软件名                        查看已安装软件的文档安装位置
rpm -qR 软件名                        查看已安装软件依赖包和文件
 
对未安装软件信息的查询
rpm -qpi rpm文件                        查看一个软件包的用途和版本信息
rpm -qpl rpm文件                        查看一个软件包所包含的文件
rpm -qpd rpm文件                        查看软件包的文档所在位置
rpm -qpc rpm文件                        查看软件包的配置文件
rpm -qpR rpm文件                        查看软件包的依赖关系
 
软件包的安装、升级、删除
rpm -ivh rpm文件                        安装rpm包
rpm -Uvh rpm文件                          更新rpm包
rpm -e 软件名                            删除rpm包
rpm -e 软件名 --nodeps                    不管依赖关系，强制删除软件
 
 导入签名
rpm --import 签名文件                   
rpm --import RPM-GPG-KEY
```
# yum
```sh
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
yum list                                   列出资源库中所有可以安装或更新的rpm包
yum list firefox*                       列出资源库中可以安装、可以更新、已安装的指定rpm包
yum list update                        列出资源库中可以更新的rpm包
yum list installed                      列出所有已安装的rpm包
yum list extras                         列出已安装但不包含在资源库中rpm包
yum groupinfo group1 显示程序组group1信息yum search string 根据关键字string查找安装包
yum info                                列出资源库中所有可以安装或更新的rpm包信息
yum info firefox*                       列出资源库可以安装或更新的指定的rpm的信息
yum info update                         列出资源库中可以更新的rpm包信息
yum info installed                      列出已安装的所有rpm包信息
yum info extras                         列出已安装到时不包含在资源库中rpm包信息
 
yum search firefox                      搜索匹配特定字符的rpm包
yum provides firefox                    搜索包含特定文件的rpm包

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

# firewall
```sh
使用自带防火墙：CentOS7使用的是Linux Kernel 3.10.0的内核版本，新版的Kernel内核已经有了防火墙netfilter
方法一
cp /usr/lib/firewalld/services/http.xml /etc/firewalld/services/
firewall-cmd --reload
方法二 间接修改/etc/firewalld/zones/public.xml文件
##Add
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-service=ftp
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
# 常用命令
```sh
tail -f /var/log/secure 查看日志
ps -aux | grep rstudio
netstat -lnp|grep 88
```
