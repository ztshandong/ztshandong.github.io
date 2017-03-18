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
yum install packagename
yum update
yum remove packagename
```
