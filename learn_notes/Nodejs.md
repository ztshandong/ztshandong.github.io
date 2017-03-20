# CentOS7安装nodejs
```sh
方法一：（推荐）
yum -y install gcc gcc-c++ kernel-devel
查看最新版本
https://nodejs.org/zh-cn/download/current/
wget https://nodejs.org/dist/v7.7.3/node-v7.7.3.tar.gz
tar -xf node-v4.5.0.tar.gz
rm -f node-v4.5.0.tar.gz
cd node-v4.5.0
./configure
make
sudo make install
方法二：
su root
curl -sL https://rpm.nodesource.com/setup | bash -
yum install -y nodejs  这样nodejs版本是0.x
方法三：
NVM（Node version manager）顾名思义，就是Node.js的版本管理软件，可以轻松的在Node.js各个版本间切换，
项目源码[GitHub](https://github.com/creationix/nvm)貌似只剩iojs了
先去github上查看最新版本
source ~/.bash_profile
nvm install node 安装最新版
```




# mac
# 安装homebrew
```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
# 安装nodejs
```sh
brew install nodejs
node --version
brew install -g n  升级工具
sudo n latest
```

# 安装mongodb
```sh
brew install mongodb
默认27017端口
```
# 安装redis
```sh
brew install redis
```
### 启动redis
```sh
redis-server  可启动，但是按ctrl+c或者关闭终端就关闭了redis
vim etc/redis.conf     输入?进入查找模式，将daemonize改为yes
redis-server /etc/redis.conf 可将redis以守护进程的方式启动
ps aux | grep redis 
redis-cli 可连接，默认6379
```
# 安装sublime text 3 3126
```sh
—– BEGIN LICENSE —–
Ryan Clark
Single User License
EA7E-812479
2158A7DE B690A7A3 8EC04710 006A5EEB
34E77CA3 9C82C81F 0DB6371B 79704E6F
93F36655 B031503A 03257CCC 01B20F60
D304FA8D B1B4F0AF 8A76C7BA 0FA94D55
56D46BCE 5237A341 CD837F30 4D60772D
349B1179 A996F826 90CDB73C 24D41245
FD032C30 AD5E7241 4EAA66ED 167D91FB
55896B16 EA125C81 F550AF6B A6820916
—— END LICENSE ——
```
# 安装sublime text 3 package control
```sh
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
# sublime text 安装 sftp
