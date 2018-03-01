# AzureUbuntu16.04挂载新磁盘
```sh
ls -l /dev/sd*    输出结果/dev/sdc一般就是新磁盘

sudo fdisk /dev/sdc    分区
Command (m for help): n   是创建新分区，输入p然后大小默认
Command (m for help): p   是显示分区状态
Command (m for help): w   是写入分区表

分区之后会有/dev/sdc1
sudo mkfs -t ext4 /dev/sdc1   转换为文件系统

sudo mkdir newdisk1  创建挂载的文件夹
sudo mount /dev/sdc1 ~/newdisk1   挂载上来

sudo -i blkid   记录UUID
sudo vi /etc/fstab
UUID=12991430-c8f7-4d30-9da7-8da225c000bb  ~/newdisk1   ext4   defaults,nofail   1   2

验证
sudo umount newdisk1
sudo mount newdisk1

sudo chmod go+w ~/newdisk1/
```

# 安装卸载
```sh
apt-get install可以安装什么类型的文件？dpkg -i呢？
两者的区别是dpkg绕过apt包管理数据库对软件包进行操作，所以你用dpkg安装过的软件包用apt可以再安装一遍，系统不知道之前安装过了，将会覆盖之前dpkg的安装。
1、dpkg是用来安装.deb文件,但不会解决模块的依赖关系,且不会关心ubuntu的软件仓库内的软件,可以用于安装本地的deb文件。
2、apt会解决和安装模块的依赖问题,并会咨询软件仓库, 但不会安装本地的deb文件, apt是建立在dpkg之上的软件管理工具。
安装软件包
dpkg -i package_name.deb              //安装本地软件包，不解决依赖关系
apt-get install package                    //在线安装软件包
aptitude install pattern                    //同上
apt-get install package --reinstall   //重新安装软件包
apitude reinstall package                //同上
移除软件包
dpkg -r package                               //删除软件包
apt-get remove package                  //同上
aptitude remove package                //同上
dpkg -P                                             //删除软件包及配置文件
apt-get remove package --purge    //删除软件包及配置文件
apitude purge pattern                      //同上
自动移除软件包
apt-get autoremove                        //删除不再需要的软件包

安装 make && make install的软件使用auto-apt 和 checkinstall，方便卸载
建议使用--prefix=/path/you/want/to/install/  来编译安装，卸载时只需要删除目录即可
常见的反安装target有： make uninstall/distclean/veryclean 等等。如果没有， 
事先记得记录'make install'的所有输出日志，'make install &> |tee make.log'。
如果有rpm包的话尽量避免使用源码包安装，linux的rpm包管理工具很强大，方便管理。如果必须要用源码包安装，请在安装的时候指定--prefix安装目录，另外安装的时候请使用make >& LOG_make &make install >& LOG_install &

一般来说，make install完了目录下会有一个install_mainfest.txt的文件记录了安装的所有内容，
然后xargs rm < install_manifest.txt就可以了。

sudo apt install -y auto-apt checkinstall
tar -zxvf xxx.gz
auto-apt run ./configure --prefix=/path
make
sudo checkinstall
3 - Version需要改为数字
```

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
# 安装icc
```sh
sudo vi ~/.bashrc
export PATH=$PATH:/opt/intel/bin

source ~/.bashrc

PATH就是whereis icc的路径
```
# C++编译调用so文件
```c++
so文件为动态链接库文件，与windows下的dll文件相当，linux下系统so文件一般保存在/usr/lib中。

==========test.h===========
#ifdef __cplusplus //
extern "C"
{
#endif
  
class Test{
public:
int hello(int i);
};
int helloT(int j);
#ifdef __cplusplus
}
#endif 
 
==========test.cpp===========
#include"test.h"
#include<iostream>
using namespace std;
int Test::hello(int i){
       if(i>3)
                 cout<<"hello Class Test>3"<<endl;
       else
                  cout<<"hello Class Test<3"<<endl;
        return 0;
}
int helloT(int j){
      Test *t=new Test();
       t->hello(j);
       return 0;
}
 
编译test.cpp文件
g++ -shared -fpic -lm -ldl -o libtest.so test.cpp
其中，so文件名必须以lib开头。编译具体指令请参考帮助文档
 
==========main.cpp===========
#include <stdio.h>   
#include <dlfcn.h>
#include <stdlib.h>   
#include <iostream>   
using namespace std;  
/*
需要用到的函数
dlopen()
dlerror()
dlsym()
dlclose()
都存储在头文件dlfcn.h中
*/
int main()  {  
      
    void *handle = dlopen("./libtest.so", RTLD_LAZY);  //该处的./libtest.so表示so文件的存放位置，RTLD_LAZY是指示位
    if(!handle)  {  
        printf("open lib error\n");  
        cout<<dlerror()<<endl;  
        return -1;  
    }  
      
     typedef int (*hello)(int);//该处的函数与文件test.h中需调用的函数保持一致
     hello h= (hello)dlsym(handle, "helloT");// helloT为test.h中调用函数的名字，dlsym返回一个函数指针
     if(!h)  {  
        cout<<dlerror()<<endl; 
        dlclose(handle);  
        return -1;  
    }  
      
     int i;
     cin>>i;
    (*h)(i);//用函数指针形式调用函数
    dlclose(handle);  
    return 0;  
}  
编译main.cpp文件
g++ main.cpp -ldl -o main
执行./main
```
# 编译TBB
```c++
sudo apt-get install -y libtbb-dev
//vi tbbvars.sh
//export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/compilers_and_libraries_2018.0.128/linux/tbb/lib/ia32_lin/gcc4.4/

testsort.cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <cassert>
#include <chrono>
#include <iomanip>
#include <tbb/task_scheduler_init.h>
#include <tbb/parallel_sort.h>

using namespace std;

const int SIZE = 10000000;

#define TIME_STD(X) { \
    auto t0 = chrono::high_resolution_clock::now(); \
    {X;} \
    auto t1 = chrono::high_resolution_clock::now(); \
    cout << setw(10) << fixed << (double)chrono::duration_cast<chrono::nanoseconds>(t1-t0).count() / (double)1000000000 << "ms " << #X << endl; \
}

int main(int argc, char* argv[])
{
    vector<int> vec_int(SIZE);
    iota(begin(vec_int), end(vec_int), 0);
    srand(0);
    random_shuffle(begin(vec_int), end(vec_int));
    
    //TIME_STD(sort(begin(vec_int), end(vec_int)));
    TIME_STD(tbb::task_scheduler_init _; tbb::parallel_sort(begin(vec_int), end(vec_int)));
    assert(is_sorted(begin(vec_int), end(vec_int)));

    return 0;
}
g++ testsort.cpp -o testsort  -I /opt/intel/tbb/include -ltbb -std=c++11






#include "tbb/parallel_for.h"
#include "tbb/task_scheduler_init.h"
#include <iostream>
#include <vector>

struct mytask {
  mytask(size_t n)
    :_n(n)
  {}
  void operator()() {
    for (int i=0;i<1000000;++i) {}  // Deliberately run slow
    std::cerr << "[" << _n << "]";
  }
  size_t _n;
};

int main(int,char**) {

  //tbb::task_scheduler_init init;  // Automatic number of threads
  tbb::task_scheduler_init init(tbb::task_scheduler_init::default_num_threads());  // Explicit number of threads

  std::vector<mytask> tasks;
  for (int i=0;i<1000;++i)
    tasks.push_back(mytask(i));

  tbb::parallel_for(
    tbb::blocked_range<size_t>(0,tasks.size()),
    [&tasks](const tbb::blocked_range<size_t>& r) {
      for (size_t i=r.begin();i<r.end();++i) tasks[i]();
    }
  );

  std::cerr << std::endl;

  return 0;
}
g++ -std=c++11 tbb_example.cpp -ltbb -o tbb_example
```







#include <cstdlib>
#include <cmath>
#include <complex>
#include <ctime>
#include <iostream>
#include <iomanip>
#include "tbb/tbb.h"
#include "tbb/blocked_range.h"
#include "tbb/parallel_for.h"
#include "tbb/parallel_for_each.h"
#include "tbb/task_scheduler_init.h"

using namespace std;
using namespace tbb;
typedef complex<double> dcmplx;

dcmplx random_dcmplx ( void )
{
   double e = 2*M_PI*((double) rand())/RAND_MAX;
   dcmplx c(cos(e),sin(e));
   return c;
}

class ComputePowers
{
   vector<dcmplx>  c; // numbers on input
   int d;           // degree
   mutable vector<dcmplx>  result;  // output
   public:
      ComputePowers(vector<dcmplx> x, int deg, vector<dcmplx> y): c(x), d(deg), result(y) { }

      void operator() ( blocked_range<size_t>& r ) const
      {
         for(int i=r.begin(); i!=r.end(); ++i)
         {
            dcmplx z(1.0,0.0);
            for(int j=0; j < d; j++) {
                z = z*c[i];
            };
            result[i] = z;
         }
      }
};

int main ()
{
   int deg = 100;
   int dim = 10;

   vector<dcmplx> r;
   for(int i=0; i<dim; i++)
     r.push_back(random_dcmplx());

   vector<dcmplx> s(dim);

   task_scheduler_init init(task_scheduler_init::automatic);

   parallel_for(blocked_range<size_t>(0,dim),
                ComputePowers(r,deg,s));
   for(int i=0; i<dim; i++)
       cout << scientific << setprecision(4)
       << "x[" << i << "] = ( " << s[i].real()
       << " , " << s[i].imag() << ")\n";
   return 0;
}
g++ -std=c++11 test1.cpp -ltbb -o test1
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
# 安装docker，感谢阿里云
```sh
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]


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