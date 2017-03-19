# CentOS7
```sh
安装基本库
yum -y install gcc gcc-c++
gcc --version
yum -y install ncurses-devel
yum -y install readline-devel
yum -y install libaio-devel

安装Development Tools
http://vault.centos.org/6.6/SCL/x86_64/scl-utils/  centos的包网站
cd /etc/yum.repos.d/
yum -y install wget
wget https://copr.fedoraproject.org/coprs/hhorak/devtoolset-4-rebuild-bootstrap/repo/epel-7/hhorak-devtoolset-4-rebuild-bootstrap-epel-7.repo -O /etc/yum.repos.d/hhorak-devtoolset-4-rebuild-bootstrap-epel-7.repo
yum --disablerepo='*' --enablerepo='hhorak-devtoolset-4-rebuild-bootstrap' list
yum -y install centos-release-scl
yum -y install scl-utils-build
yum --disablerepo="*" --enablerepo="scl" list available 
yum group list 
yum group install "Development Tools" 会报错，提示找不到DevelopmentTools，用下面的命令
yum -y --setopt=group_package_types=mandatory,default,optional groupinstall "Development Tools"
安装scl-utils-20120927-11
http://rpm.pbone.net/index.php3/stat/4/idpl/26383700/dir/scientific_linux_6/com/scl-utils-20120927-11.el6_5.x86_64.rpm.html
wget ftp://mirror.switch.ch/pool/4/mirror/scientificlinux/6.5/x86_64/updates/fastbugs/scl-utils-20120927-11.el6_5.x86_64.rpm
rpm -ivh scl-utils-20120927-11.el6_5.x86_64.rpm
下面这条命令必须要scl-utils-20120927-11这个版本，并且下载速度很慢，蛋疼
yum -y --disablerepo='*' --enablerepo='hhorak-devtoolset-4-rebuild-bootstrap' install devtoolset-4-gcc devtoolset-4-gcc-c++
scl enable devtoolset-4 bash
gcc --version

```
