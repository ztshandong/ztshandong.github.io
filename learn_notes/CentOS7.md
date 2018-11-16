# 安装
```sh
建议安装英文
Add-Ons中选择Development Tools即可，其他不用选
选择硬盘界面虽然默认已选，但最好是先取消再重新选择
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
#### 安装Vmtools
```sh
若是未安装图形界面貌似不用装这个
cp vmtools.gz /tmp
cd /tmp
tar -zxf vmtools.gz
cd vm-tools
su root
./vmware-install.pl -d
/usr/bin/vmware-config-tools.pl
vmware-hgfsclient
vmhgfs-fuse .host:/D /mnt/hgfs
```
# 安装JDK，首先卸载自带java
```sh
sudo yum -y remove java    
sudo rpm -ivh jdk-7u80-linux-x64.rpm
```
# 安装docker
```sh
以前只要一句话，现在好像不行了
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://0rnhdnox.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl restart docker
```
# [安装vscode](https://code.visualstudio.com/docs/setup/linux)
```sh
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
yum check-update
sudo yum install -y code

```
# network
```centos
centos网卡默认不启用
sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 (网卡名可能不同)
ONBOOT=yes
然后重启网卡
systemctl restart network
```
# yum提速
- /etc/yum.conf

#### 方法一，推荐阿里的
```sh
http://mirrors.aliyun.com/repo/
su root
yum -y install wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/epel-7.repo
yum clean all
yum makecache
yum install -y deltarpm
yum -y update

yum -y install epel-release 再安装epel仓库提示重复
rpm -Uvh https://centos7.iuscommunity.org/ius-release.rpm ius仓库，可选
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm remi仓库，可选

wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
mv CentOS6-Base-163.repo /etc/yum.repos.d/CentOS-Base.repo

yum clean all
yum makecache
yum -y update
```
#### 方法二  fastestmirror 插件
```sh
yum install yum-fastestmirror -y
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.old
```
#### 方法三 fastestmirror和axelget插件
```sh
yum install yum-fastestmirror -y
yum -y install gcc make
yum install -y axel
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
yum -y install policycoreutils-python 未安装就安装
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

# ftp
```
ftp
open 192.168.1.250
close
!dir   dir
!
bin
put xxx
get xxx
delete xxx
cd ..
bye

vi /etc/selinux/config
SELINUX=disabled
setenforce 0 

sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 (网卡名可能不同)
ONBOOT=yes

vi /etc/vsftpd/vsftpd.conf
anonymous_enable=NO 
local_enable=YES
write_enable=YES
chroot_local_user=YES
userlist_enable=NO

vi /etc/vsftpd/user_list
ftpuser 

mkdir /ftpuser
useradd ftpuser -s /sbin/nologin -d /ftpuser
passwd ftpuser

systemctl restart vsftpd
systemctl enable vsftpd
systemctl restart vsftpd

setenforce 1
/usr/sbin/sestatus -v
getsebool -a | grep ftp
setsebool -P tftp_home_dir 1
setsebool -P allow_ftpd_full_access 1

FTP命令是Internet用户使用最频繁的命令之一，不论是在DOS还是UNIX操作系统下使用FTP，都会遇到大量的FTP内部命令。熟悉并灵活应用FTP的内部命令，可以大大方便使用者，并收到事半功倍之效。
FTP的命令行格式为：ftp -v -d -i -n -g [主机名]，其中
-v显示远程服务器的所有响应信息；
-n限制ftp的自动登录，即不使用；
.n etrc文件；
-d使用调试方式；
-g取消全局文件名。
ftp使用的内部命令如下(中括号表示可选项):
1.![cmd[args]]：在本地机中执行交互shell，exit回到ftp环境，如：!ls*.zip.
2.$ macro-ame[args]：执行宏定义macro-name.
3.account[password]：提供登录远程系统成功后访问系统资源所需的补充口令。
4.append local-file[remote-file]：将本地文件追加到远程系统主机，若未指定远程系统文件名，则使用本地文件名。
5.ascii：使用ascii类型传输方式。
6.bell：每个命令执行完毕后计算机响铃一次。
7.bin：使用二进制文件传输方式。
8.bye：退出ftp会话过程。
9.case：在使用mget时，将远程主机文件名中的大写转为小写字母。
10.cd remote-dir：进入远程主机目录。
11.cdup：进入远程主机目录的父目录。
12.chmod mode file-name：将远程主机文件file-name的存取方式设置为mode，如：chmod 777 a.out。
13.close：中断与远程服务器的ftp会话(与open对应)。
14.cr：使用asscii方式传输文件时，将回车换行转换为回行。
15.delete remote-file：删除远程主机文件。
16.debug[debug-value]：设置调试方式，显示发送至远程主机的每条命令，如：deb up 3，若设为0，表示取消debug。
17.dir[remote-dir][local-file]：显示远程主机目录，并将结果存入本地文件local-file。
18.disconnection：同close。
19.form format：将文件传输方式设置为format，缺省为file方式。
20.get remote-file[local-file]：将远程主机的文件remote-file传至本地硬盘的local-file。
21.glob：设置mdelete，mget，mput的文件名扩展，缺省时不扩展文件名，同命令行的-g参数。
22.hash：每传输1024字节，显示一个hash符号(#)。
23.help[cmd]：显示ftp内部命令cmd的帮助信息，如：help get。
24.idle[seconds]：将远程服务器的休眠计时器设为[seconds]秒。
25.image：设置二进制传输方式(同binary)。
26.lcd[dir]：将本地工作目录切换至dir。
27.ls[remote-dir][local-file]：显示远程目录remote-dir，并存入本地文件local-file。
28.macdef macro-name：定义一个宏，遇到macdef下的空行时，宏定义结束。
29.mdelete[remote-file]：删除远程主机文件。
30.mdir remote-files local-file：与dir类似，但可指定多个远程文件，如：mdir *.o.*.zipoutfile
31.mget remote-files：传输多个远程文件。
32.mkdir dir-name：在远程主机中建一目录。
33.mls remote-file local-file：同nlist，但可指定多个文件名。
34.mode[modename]：将文件传输方式设置为modename，缺省为stream方式。
35.modtime file-name：显示远程主机文件的最后修改时间。
36.mput local-file：将多个文件传输至远程主机。
37.newer file-name：如果远程机中file-name的修改时间比本地硬盘同名文件的时间更近，则重传该文件。
38.nlist[remote-dir][local-file]：显示远程主机目录的文件清单，并存入本地硬盘的local-file。
39.nmap[inpattern outpattern]：设置文件名映射机制，使得文件传输时，文件中的某些字符相互转换，如：nmap $1.$2.$3[$1，$2].[$2，$3]，则传输文件a1.a2.a3时，文件名变为a1，a2。该命令特别适用于远程主机为非UNIX机的情况。
40.ntrans[inchars[outchars]]：设置文件名字符的翻译机制，如ntrans 1R，则文件名LLL将变为RRR。
41.open host[port]：建立指定ftp服务器连接，可指定连接端口。
42.passive：进入被动传输方式。
43.prompt：设置多个文件传输时的交互提示。
44.proxy ftp-cmd：在次要控制连接中，执行一条ftp命令，该命令允许连接两个ftp服务器，以在两个服务器间传输文件。第一条ftp命令必须为open，以首先建立两个服务器间的连接。
45.put local-file[remote-file]：将本地文件local-file传送至远程主机。
46.pwd：显示远程主机的当前工作目录。
47.quit：同bye，退出ftp会话。
48.quote arg1，arg2...：将参数逐字发至远程ftp服务器，如：quote syst.
49.recv remote-file[local-file]：同get。
50.reget remote-file[local-file]：类似于get，但若local-file存在，则从上次传输中断处续传。
51.rhelp[cmd-name]：请求获得远程主机的帮助。
52.rstatus[file-name]：若未指定文件名，则显示远程主机的状态，否则显示文件状态。
53.rename[from][to]：更改远程主机文件名。
54.reset：清除回答队列。
55.restart marker：从指定的标志marker处，重新开始get或put，如：restart 130。
56.rmdir dir-name：删除远程主机目录。
57.runique：设置文件名唯一性存储，若文件存在，则在原文件后加后缀..1，.2等。
58.send local-file[remote-file]：同put。
59.sendport：设置PORT命令的使用。
60.site arg1，arg2...：将参数作为SITE命令逐字发送至远程ftp主机。
61.size file-name：显示远程主机文件大小，如：site idle 7200。
62.status：显示当前ftp状态。
63.struct[struct-name]：将文件传输结构设置为struct-name，缺省时使用stream结构。
64.sunique：将远程主机文件名存储设置为唯一(与runique对应)。
65.system：显示远程主机的操作系统类型。
66.tenex：将文件传输类型设置为TENEX机的所需的类型。
67.tick：设置传输时的字节计数器。
68.trace：设置包跟踪。
69.type[type-name]：设置文件传输类型为type-name，缺省为ascii，如：type binary，设置二进制传输方式。
70.umask[newmask]：将远程服务器的缺省umask设置为newmask，如：umask 3。
71.user user-name[password][account]：向远程主机表明自己的身份，需要口令时，必须输入口令，如：user anonymous my@email。
72.verbose：同命令行的-v参数，即设置详尽报告方式，ftp服务器的所有响应都将显示给用户，缺省为on.
73.?[cmd]：同help。

      假设FTP地址为“ 61.129.83.39”（大家试验的时候不要以这个FTP去试，应该可能密码要改掉。）
      1：“开始”-“运行”-输入“FTP”进去cmd界面
      2.open    61.129.83.39
      如果你的FTP服务器不是用的21默认端口，假如端口是9900，那么此步的命令应在后面空格加9900，即为 open 61.129.83.39    9900
      3：它会提示输入用户名 username
      4: 它会提示你输入密码：password　　　　　
      注意密码不显示出来，打完密码后回车即可。如果你的密码输入错误，将不会提示你重新输入，这时你只要键入“user”命令，你就可以重新输入用户名和密码。
      5：成功登陆后就可以用dir查看命令查看FTP服务器中的文件及目录，用ls命令只可以查看文件。
      6：使用cd 命令转目录,delete删文件，用法跟DOS差不多。呵呵！！
      7：lcd d:dianying 定位本地默认文件夹（本人理解这里的L是local当地英文的缩写，很好理解和记忆）
      8：下面就是上传和下载文件的命令了，上传用put 文件名.下载用get 文件名
      当然下载到当前目录了，就是上面定义的"d:dianying"
      9：最后就退出了
      用bye命令。
ftp [-v][-d][-i][-n][-g][-s:FileName][-a][-w:WindowSize][-A][Host]

参数
-v 
   禁止显示 FTP 服务器响应。 
/d 
   启用调试、显示在 FTP 客户端和 FTP 服务器之间传递的所有命令。 
-i 
   传送多个文件时禁用交互提示。 
-n 
   在建立初始连接后禁止自动登录功能。 
-g 
   禁用文件名组合。Glob 允许使用星号 (*) 和问号 (?) 作为本地文件和路径名
的通配符字符。
-s:filename 
   指定包含 ftp 命令的文本文件。这些命令在启动 ftp 后自动运行。该参数不
允许带有空格。使用该参数而不是重定向 (<)。 
-a 
   指定绑定 FTP 数据连接时可以使用任何本地接口。 
-w:windowsize 
   指定传输缓冲的大小。默认窗口大小为 4096 字节。 
-A 
   匿名登录到 FTP 服务器。 
Host 
   指定要连接的计算机名、IP 地址或 FTP 服务器的 IPv6 地址。如果指定了主
机名或地址，则其必须是命令行的最后一个参数。 
/? 
   在命令提示符下显示帮助。
　常用命令:    　　　
1. open：与ftp服务器相连接； 
2. send(put)：上传文件； 
3. get：下载文件； 
4. mget：下载多个文件； 
5. cd：切换目录；

```

# keepalive
```
/proc/sys/net/ipv4

vi tcp_keepalive_intvl 
vi tcp_keepalive_probes 
vi tcp_keepalive_time 

```
# teamviewer
```
https://pkgs.org/download/libQt5WebKitWidgets.so.5%28%29%2864bit%29
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/q/qt5-qtwebkit-5.9.1-1.el7.x86_64.rpm

systemctl start teamviewerd
teamviewer_13.2.13582.x86_64.rpm
teamviewer --setup console #设置启动方式为控制台启动  
teamviewer --daemon restart  #重启teamview服务  
teamviewer --info #查看teamview信息  
teamviewer --passwd [PASSWD]   #设置密码  
teamviewer --help  #查看帮助  

```

# 常用命令
```sh
source /etc/profile
tail -f /var/log/secure 查看日志
ps -aux | grep rstudio
netstat -lnp|grep 88
find / -name mysql.soc
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock 创建快捷方式
cat /etc/group
cat /etc/passwd
usermod -d /home/test -G test2 test
将test用户的登录目录改成/home/test，并加入test2组，注意这里是大G。
gpasswd -a test test2 将用户test加入到test2组
gpasswd -d test test2 将用户test从test2组中移出

a），查看当前登录用户
[root@krlcgcms01 ~]# w
[root@krlcgcms01 ~]# who
b），查看自己的用户名
[root@krlcgcms01 ~]# whoami
c），查看单个用户信息
[root@krlcgcms01 ~]# finger apacheuser
[root@krlcgcms01 ~]# id apacheuser
d），查看用户登录记录
[root@krlcgcms01 ~]# last 查看登录成功的用户记录
[root@krlcgcms01 ~]# lastb 查看登录不成功的用户记录
e），查看所有用户
[root@krlcgcms01 ~]# cut -d : -f 1 /etc/passwd
[root@krlcgcms01 ~]# cat /etc/passwd |awk -F \: '{print $1}'
```
