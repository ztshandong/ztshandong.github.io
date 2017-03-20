# 安装R语言
#### [安装zlib](http://www.zlib.net/)
#### [安装bzip2](http://www.bzip.org/downloads.html)
#### [安装liblzma](http://tukaani.org/xz/)
#### [安装pcre](https://ftp.pcre.org/pub/pcre/)
```sh
yum remove pcre
wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz
./configure --enable-utf8 要启用utf8
```
#### [安装libcurl](https://curl.haxx.se/libcurl/)

#### [下载R](https://www.r-project.org/)
```sh
./configure --with-x=no  如果没安装图形界面需要此参数
```


