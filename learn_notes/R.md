# CentOS7安装R语言
#### [安装zlib](http://www.zlib.net/)
#### [安装bzip2](http://www.bzip.org/downloads.html)
#### [安装liblzma](http://tukaani.org/xz/)
#### [安装pcre](https://ftp.pcre.org/pub/pcre/)
```sh
./configure --enable-utf8 
yum remove pcre 默认安装的可能没有启用uft8，重新安装，顺便安装个高级点的
```
#### [安装libcurl](https://curl.haxx.se/libcurl/)

#### [下载R](https://www.r-project.org/)
```sh
方法一：
yum install epel-release
yum install R
方法二：
yum -y install readline-devel    不用--with-readline=no
./configure --with-x=no  --enable-R-shlib  如果没安装图形界面需要此参数
已安装过要先清除
make clean
make uninstall
```
#### [安装RStudio](https://www.rstudio.com/)
```sh
我安装的是Server版
yum -y install --nogpgcheck rstudio-server-rhel-1.0.136-x86_64.rpm
rstudio-server verify-installation
```
