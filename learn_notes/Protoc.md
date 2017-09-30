# Ubuntu安装
```sh
https://github.com/google/protobuf/releases/

wget https://github.com/google/protobuf/releases/download/v3.4.1/protobuf-cpp-3.4.1.tar.gz
tar -zxvf protobuf-cpp-3.4.1.tar.gz
cd protobuf-3.4.1
./configure
make
make check
sudo make install

protobuf的默认安装路径是/usr/local/lib，而/usr/local/lib 不在Ubuntu体系默认的 LD_LIBRARY_PATH 里，所以就找不到该lib 
sudo vi /etc/ld.so.conf.d/libprotobuf.conf
/usr/local/lib

sudo ldconfig
```