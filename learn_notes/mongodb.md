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
