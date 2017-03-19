# CentOS7
```sh
yum -y install gcc gcc-c++
gcc --version
yum -y install ncurses-devel
yum -y install readline-devel
yum -y install libaio-devel

yum group list 
cd /etc/yum.repos.d/
yum -y install wget
wget https://copr.fedoraproject.org/coprs/hhorak/devtoolset-4-rebuild-bootstrap/repo/epel-7/hhorak-devtoolset-4-rebuild-bootstrap-epel-7.repo -O /etc/yum.repos.d/hhorak-devtoolset-4-rebuild-bootstrap-epel-7.repo
yum --disablerepo='*' --enablerepo='hhorak-devtoolset-4-rebuild-bootstrap' list
yum -y install centos-release-scl
yum -y install scl-utils-build
yum --disablerepo="*" --enablerepo="scl" list available 
yum --disablerepo='*' --enablerepo='hhorak-devtoolset-4-rebuild-bootstrap' install devtoolset-4-gcc devtoolset-4-gcc-c++
 scl enable devtoolset-4 bash
 gcc --version
```
