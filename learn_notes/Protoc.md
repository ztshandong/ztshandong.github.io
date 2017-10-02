# Ubuntu安装
```sh
https://github.com/google/protobuf/releases/

Linux
sudo apt-get install -y build-essential autoconf libtool
sudo apt-get install -y libgflags-dev libgtest-dev
sudo apt-get install -y clang libc++-dev
sudo apt-get install -y pkg-config

macOS
sudo xcode-select --install
brew install autoconf automake libtool shtool
brew install gflags

Ubuntu下只用过c++的，后面grpc要用which grpc_cpp_plugin
https://github.com/google/protobuf/releases/download/v3.4.1/protobuf-cpp-3.4.1.tar.gz
tar -zxvf v3.4.1.tar.gz
cd protobuf-3.4.1

如果下的是https://github.com/google/protobuf/archive/v3.4.1.tar.gz就要先./autogen.sh

./configure

MacOS好像要用LIBTOOL=glibtool LIBTOOLIZE=glibtoolize make
make
make check
sudo make install

protoc --version

protobuf的默认安装路径是/usr/local/lib，而/usr/local/lib 不在Ubuntu体系默认的 LD_LIBRARY_PATH 里，所以就找不到该lib 
sudo vi /etc/ld.so.conf.d/libprotobuf.conf
/usr/local/lib

sudo ldconfig

MacOS貌似brew install protobuf即可

git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
make
sudo make install
```