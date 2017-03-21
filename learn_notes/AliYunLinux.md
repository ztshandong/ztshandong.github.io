Aliyun Linux是RHEL(RedHat Enterprise linux)再编译的产物。RHEL系统本身是免费的，但是RHEL官方的yum源是收费的，需要REDHAT公司的技术支持的话也是收费的，一般使用RHEL的系统也都是些企业用户奔着收费技术支持来的。 REDHAT公司允许去除RHEL的相关标识后二次编译RHEL的源码后再次发行，比较有名的产物是CENTOS，阿里云自己也搞了个"Aliyun Linux"，基本类似。
由于RedHat的yum在线更新是收费的，我们的RedHat没有注册，不能在线更新下载rpm包。
需将aliyun linux 5.7的yum卸载后，重启安装Centos的yum,再配置其他yum源。
 
一、确认aliyun的版本
cat /etc/aliyun-release
uname -m
------
[root@iZ28z295fu9Z /]# cat /etc/aliyun-release

Aliyun Linux release 5.7

[root@iZ28z295fu9Z /]# uname -m

x86_64

------
二、删除Aliyun linux原有的yum源
rpm -aq|grep yum|xargs rpm -e --nodeps

三、下载CentOS的yum安装包（163源）  (可以在这里手工找rpm包：http://mirrors.163.com/centos/)
64位系统：
wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-metadata-parser-1.1.2-4.el5.x86_64.rpm

wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-3.2.22-40.el5.centos.noarch.rpm

wget http://mirrors.163.com/centos/5/os/x86_64/CentOS/yum-fastestmirror-1.1.16-21.el5.centos.noarch.rpm

四、安装yum软件包
64位系统：
rpm -ivh yum-metadata-parser-1.1.2-4.el5.x86_64.rpm
rpm -ivh yum-3.2.22-40.el5.centos.noarch.rpm  yum-fastestmirror-1.1.16-21.el5.centos.noarch.rpm
注意：最后两个安装包要放在一起同时安装，否则会提示相互依赖，安装失败。

五、更改yum源  #我们使用网易的CentOS镜像源
cd /etc/yum.repos.d/
vi CentOS6-Base-163.repo
如下是文件内容：
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#
[base]
name=CentOS-5 - Base - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=os
baseurl=http://mirrors.163.com/centos/5/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
#released updates 
[updates]
name=CentOS-5 - Updates - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=updates
baseurl=http://mirrors.163.com/centos/5/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
#packages used/produced in the build but not released
[addons]
name=CentOS-5 - Addons - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=addons
baseurl=http://mirrors.163.com/centos/5/addons/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
#additional packages that may be useful
[extras]
name=CentOS-5 - Extras - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=extras
baseurl=http://mirrors.163.com/centos/5/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-5 - Plus - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=centosplus
baseurl=http://mirrors.163.com/centos/5/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5
#contrib - packages by Centos Users
[contrib]
name=CentOS-5 - Contrib - 163.com
mirrorlist=http://mirrorlist.centos.org/?release=5&arch=$basearch&repo=contrib
baseurl=http://mirrors.163.com/centos/5/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5


六、清理
yum clean all
yum makecache  
更新之后，系统会自动加上aliyun的资源镜像。
如果报错：imary.sqlite.bz2 from base: [Errno 256] No more mirrors to try.
执行：yum makecache   
 
七、更新
cd /etc/pki/rpm-gpg
wget http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5
yum update

很长时间的下载后，报错：
warning: rpmts_HdrFromFdno: Header V3 DSA signature: NOKEY, key ID e8562897
GPG key retrieval failed: [Errno 5] OSError: [Errno 2] No such file or directory: '/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5'
这是因为：指定的文件/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5在本地不存在导致的。 

解决：
去官网http://mirror.centos.org/centos/下载相应文件
cd /etc/pki/rpm-gpg
个别情况下/etc/pki下没有rpm-gpg文件夹，我们自己建立即可。
wget http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-5
