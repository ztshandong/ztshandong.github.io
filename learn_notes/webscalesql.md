# CentOS7
# 使用yum安装
```sh
vi /etc/yum.conf
[repo.psce.com_webscalesql]
name=PSCE Repository
baseurl=http://repo.psce.com/psce/yum/$releasever/$basearch
gpgkey=http://repo.psce.com/psce/RPM-GPG-KEY-PSCE
gpgcheck=1
enabled=1

yum list | grep webscalesql
yum -y install webscalesql-embedded.x86_64    安装这个版本
然后不知道该怎么办了
```

# 安装基本库
```sh
yum -y install gcc gcc-c++
gcc --version
yum -y install ncurses-devel
yum -y install readline-devel
yum -y install libaio-devel
```
# 安装Development Tools
```sh
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
# 安装软件包
```sh
cd /usr/local/src/webscale-source 没有就创建一个，下载的安装包都放这里

https://cmake.org/download/ 安装cmake
wget https://cmake.org/files/v3.8/cmake-3.8.0-rc2.tar.gz
tar -zxvf cmake-3.8.0-rc2.tar.gz
cd cmake-3.8.0-rc2
./bootstrap
make && make install

ftp://ftp.gnu.org/gnu/  安装bison与m4
wget ftp://ftp.gnu.org/gnu/bison/bison-3.0.4.tar.gz
tar -zxvf bison-3.0.4.tar.gz
cd bison-3.0.4
./configure
make && make install

wget ftp://ftp.gnu.org/gnu/m4/m4-1.4.18.tar.gz
tar -zxvf m4-1.4.18.tar.gz
cd m4-1.4.18
./configure 
make && make install

https://github.com/webscalesql 下载wescalesql
wget https://github.com/webscalesql/webscalesql-5.6/archive/webscalesql-5.6.27.zip

/usr/sbin/groupadd mysql
/usr/sbin/useradd -g mysql mysql
mkdir /data/webscalesoft -p
mkdir /data/webscaledb -p
cd /usr/local/src/webscale-source/

安装7z（可不装）
yum install -y p7zip 提示找不到，所以采取了另一种方式安装7z
https://sourceforge.net/projects/p7zip/files/
wget https://nchc.dl.sourceforge.net/project/p7zip/p7zip/16.02/p7zip_16.02_src_all.tar.bz2
tar -jxvf p7zip_16.02_src_all.tar.bz2
cd p7zip_16.02
make && make install

unzip webscalesql-5.6.27.zip
cd webscalesql-5.6-webscalesql-5.6.27
mkdir source_downloads

https://github.com/google/googlemock 下载gmock
wget https://github.com/google/googlemock/archive/master.zip
unzip master.zip gmock下载的包名字是master.zip
mv master.zip gmock-1.6.0.zip
cp gmock-1.6.0.zip /usr/local/src/webscale-source/webscalesql-5.6-webscalesql-5.6.27/source_downloads
--cp -r googlemock-master  webscalesql-5.6-webscalesql-5.6.27/source_downloads

cd  webscalesql-5.6-webscalesql-5.6.27
mysql-5.5 以上版本需要依赖于如下包文件(如果是低于 mysql-5.5 版本不需要安装 cmake )
yum -y install gcc gcc-c++ openssl openssl-devel ncurses ncurses-devel cmake
cmake -DCMAKE_INSTALL_PREFIX=/data/webscalesoft -DMYSQL_UNIX_ADDR=/data/webscalesoft/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DEXTRA_CHARSETS=all -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/data/webscaledb -DMYSQL_TCP_PORT=3308 -DENABLE_DOWNLOADS=1

Library webscalesqlclient depends on OSLIBS -lpthread;m;rt;dl  不知道怎么办

make && make install
cd /data/webscalesoft
./scripts/mysql_install_db --user=mysql --ldata=/data/webscaledb/  --defaults-file=/data/webscalesoft/my.cnf
vi my.cnf 最后添加一行
socket=/var/run/mysqld/mysqld.sock
----------------
这个方法先不用，可能之前装过mysql的配置文件，默认的文件名是my.cnf
rz my-3307.cnf
./scripts/mysql_install_db	--user=mysql	--ldata=/data/webscaledb/	-- explicit_defaults_for_timestamp --defaults-file=/data/webscalesoft/my-3307.cnf

将 mysqld_safe_3307 改名为 mysqld_safe 并替换掉/ data/webscalesoft/bin 下的 mysqld_safe
cd /data/webscalesoft/bin
rz mysqld_safe_3307
mv mysqld_safe_3307 mysqld_safe
----------------
chmod    +x    mysqld_safe

将目录/data/webscalesoft 授权给 mysql 用户：
cd ..
chown -R mysql.mysql /data/webscalesoft
chown -R mysql.mysql /data/webscaledb

在文件~/.bashrc 的最后添加如下三行:(注意将my-3307.cnf改为my.cnf)
vi    ~/.bashrc
alias	mysql3307_start="/data/webscalesoft/bin/mysqld_safe --defaults-file=/data/webscalesoft/my-3307.cnf -P 3307 -umysql&"
alias mysql3307_stop="/data/webscalesoft/bin/mysqladmin -S /data/webscalesoft/mysql.sock -P 3307 shutdown"
alias mysql3307="/data/webscalesoft/bin/mysql -S /data/webscalesoft/mysql.sock"
source ~/.bashrc

在文件/etc/profile 最后添加如下内容：
echo “export PATH=/data/webscalesoft/bin:/usr/bin:/sbin:\$PATH”>>/etc/profile
source /etc/profile

以后,
输入 mysql3307_start 时，就可以启动数据库； 
输入 mysql3307_stop 时，就可以关闭数据库； 
输入 mysql3307 时,就直接进行相应的数据库中.
随系统服务器启动的时启动数据库,可以通过如下操作：
vi /etc/rc.local        在最下面添加如下内容
/data/webscalesoft/bin/mysqld_safe --defaults-file=/data/webscalesoft/my-3307.cnf -umysql&
此时，就可以进行启动和关闭数据的操作了。 
启动数据库：
mysql3307_start
关闭数据库：
mysql3307_stop

删除默认空密码用户：
mysql> drop user ''@'webscale-01';
mysql> drop user ''@'localhost';
mysql> drop user 'root'@'::1'; 
mysql> drop user 'root'@'webscale-01'; 

```
