# 默认不需要用户名与密码即可访问
# CentOS7-MongoDB
```sh
https://www.mongodb.com/download-center?jmp=nav#community 查看版本
https://www.mongodb.org/static/pgp/ 
http://repo.mongodb.org/yum/redhat/7Server/mongodb-org/
more /etc/redhat-release 
添加源，写此文时最新稳定版为3.4
vi /etc/yum.repos.d/mongodb-org-3.4.repo 
添加以下内容
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc

yum -y install mongodb-org
安装完后创建的文件
/etc/mongod.conf－MongoDB配置文件，其中包含监听端口
/var/lib/mongo－MongoDB数据保存目录
/var/log/mongodb/mongod.log－MongoDB的日志文件
启动
systemctl start mongod.service
systemctl enable mongod.service
firewall-cmd --zone=public --add-port=27017/tcp --permanent
firewall-cmd --reload

解决”transparent huge page error”警告信息：
vi /etc/rc.local
添加
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
解决 rlimits 相关的警告：
vim /etc/security/limits.d/20-nproc.conf
添加
mongod   soft  nproc   64000
重启
systemctl restart mongod
```
# 删除
```sh
systemctl stop mongod
yum erase $(rpm -qa | grep mongodb-org)
rm -rf /var/log/mongodb
rm -rf /var/lib/mongo
```
# 查看是否可用
```sh
which mongod
```
# 创建数据存储目录
```sh
mkdir -p /data/db 
-p参数代表层级创建
df -lh 检查磁盘空间
```
# 启动mongod
```sh
mongod --dbpath /data/db --port 27017  (默认27017）
ctrl+c结束
--fork 参数可以用守护进程方式启动，但必须指定日志存储位置，可使用--syslog (centos下位置/var/log/messages)
tail -f /var/log/messages可监控文件修改)
mongod --shutdown(centos)
mac  要用kill
ps aux | grep mongo
kill 626
```
# 连接
```sh
mongo 127.0.0.1:27017
show dbs      admin与local是默认的
```
# [语法大全](http://www.cnblogs.com/liuzhongfeng/p/5588680.html)
# 插入数据
```sh
use newdb   只有插入数据后才算真正创建
集合就相当于sql中的table，一般用名词的复数形式当集合名字
db.users.insert({"username":"zhangsan"});此时会创建newdb及users
```
# 查询数据
```sh
show collections 查看集合(列出所有表名)
db.users.find()  查看集合内容(查询表内容)
db.users.find().count() 链式写法
db.users.find({"username":"zhangsan"})
db.users.find({"_id":ObjectId("58c9e9f8706983f0368ebae4")})
```
# 更新数据
```sh
三个参数：条件，更新内容，是否批量（默认否）
db.users.update({"username" : "zhangsan" },{$set:{"group" : "writer"}})  $代表要修改的值，此时只修改一条数据
db.users.update({"username" : "zhangsan" },{$set:{"group" : "writer"}}，{multi:true})
save方法参数必须为_id
db.users.save({"_id" : ObjectId("58c9ebdd706983f0368ebae6"),"group":"it"}) 但是这样这条数据只剩"group":"it"
所以在用save时必须指定所有参数，否则会丢掉
```
# 删除数据
```sh
两个参数：删除条件，是否单行删除（默认否，即默认批量删除）
db.users.remove({"group":"it"}) 删除多行
db.users.remove({"group":"it"},true) 删除一行
db.users.remove({}) 删除所有，相当于truncate table
db.users.drop() 相当于drop table
```
# [视图](https://github.com/mongodb-js/mongo-views)
# [存储过程](http://www.cnblogs.com/liuzhongfeng/p/5588781.html)
# MAC
```sh
brew install mongodb
brew unlink mongodb && brew uninstall mongodb
```
