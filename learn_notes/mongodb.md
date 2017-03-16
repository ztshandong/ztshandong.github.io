# 默认不需要用户名与密码即可访问
# 查看是否可用
```sh
whick mongod
```
# 指定数据存储目录
```sh
mkdir -p /data/db 
-p参数代表层级创建
df -lh 检查磁盘空间
```
# 启动mongod
```sh
mongod --dbpath /data/db --port 27017  (默认27017）
ctrl+c结束
--fork 参数可以用守护进程方式启动，但必须指定日志存储位置，可使用--syslog (centos下位置/var/log/messages,tail -f /var/log/messages可监控文件修改)
mongod --shutdown(centos)
mac 要用kill
```
# 连接
```sh
mongo 127.0.0.1:27017
show dbs      admin与local是默认的
```
# 插入数据
```sh
use newdb   只有插入数据后才算真正创建
集合就相当于sql中的table，一般用名词的复数形式当集合名字
db.users.insert({"username":"zhangsan"});此时会创建newdb
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
