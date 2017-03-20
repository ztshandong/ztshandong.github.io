# CentOS7安装R语言
#### [安装zlib](http://www.zlib.net/)
#### [安装bzip2](http://www.bzip.org/downloads.html)
#### [安装liblzma](http://tukaani.org/xz/)
#### [安装pcre](https://ftp.pcre.org/pub/pcre/)
```sh
yum remove pcre 默认安装的可能没有启用uft8，重新安装，顺便安装个高级点的
wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz
./configure --enable-utf8 
```
#### [安装libcurl](https://curl.haxx.se/libcurl/)

#### [下载R](https://www.r-project.org/)
```sh
./configure --with-x=no  如果没安装图形界面需要此参数
```
#### [安装RStudio](https://www.rstudio.com/)
```sh
我安装的是Server版
```
