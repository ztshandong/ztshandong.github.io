# mac
# 安装homebrew
```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
# 安装nodejs
```sh
brew install nodejs
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
