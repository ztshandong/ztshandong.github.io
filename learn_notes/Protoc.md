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




Linux
sudo apt-get install build-essential autoconf libtool
sudo apt-get install libgflags-dev libgtest-dev
sudo apt-get install clang libc++-dev
sudo apt-get install -y pkg-config

macOS
sudo xcode-select --install
brew install autoconf automake libtool shtool
brew install gflags

git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
make
sudo make install
```