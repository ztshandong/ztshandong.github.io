# 安装
```sh
yum -y install redis
```
- [redis消息队列](https://my.oschina.net/voole/blog/728897)


# windows集群
```
https://github.com/MicrosoftArchive/redis/releases
https://raw.githubusercontent.com/MSOpenTech/redis/3.2/src/redis-trib.rb
https://www.cnblogs.com/lcawen/p/7059707.html
https://www.cnblogs.com/lflyq/p/6145826.html
https://github.com/ameizi/DevArticles/issues/129

安装ruby
解压rubygems-2.7.7.zip，然后ruby setup.rb  （安装 rubyGems）
切换到redis安装目录
gem install redis
gem install redis 出现 SSL Connect error时设置 SSL_CERT_FILE 这个环境变量，并指向 cacert.pem 文件。

拷贝6份redis.xx.conf重命名，修改内容
redis.windows-6879.conf

bind 127.0.0.1 ::1
port 6380
appendonly yes
appendfilename "appendonly.6879.aof"
cluster-enabled yes
cluster-config-file nodes-6879.conf
cluster-node-timeout 15000
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes
protected-mode no

安装成服务，要安装6个
redis-server --service-install redis.windows-6879.conf --service-name redis6879 
redis-server --service-start --service-name redis6879

redis-server --service-install redis.windows-6880.conf --service-name redis6880 
redis-server --service-start --service-name redis6880

redis-server --service-install redis.windows-6881.conf --service-name redis6881 
redis-server --service-start --service-name redis6881

redis-server --service-install redis.windows-6882.conf --service-name redis6882 
redis-server --service-start --service-name redis6882

redis-server --service-install redis.windows-6883.conf --service-name redis6883 
redis-server --service-start --service-name redis6883

redis-server --service-install redis.windows-6884.conf --service-name redis6884 
redis-server --service-start --service-name redis6884

redis-server --service-uninstall --service-name redis6879
redis-server --service-uninstall --service-name redis6880
redis-server --service-uninstall --service-name redis6881
redis-server --service-uninstall --service-name redis6882
redis-server --service-uninstall --service-name redis6883
redis-server --service-uninstall --service-name redis6884



控制台配置如下
bind 192.168.1.169
port 6879
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
protected-mode no
启动redis
redis-server redis.windows.conf --protected-mode no

创建集群
redis-trib.rb放到redis文件夹下
ruby redis-trib.rb create --replicas 1 192.168.1.169:6879 192.168.1.169:6880 192.168.1.169:6881 192.168.1.169:6882 192.168.1.169:6883 192.168.1.169:6884

客户端连接
-c代表连接到集群
redis-cli.exe -c -h 192.168.1.169 -p 6879 --raw
set cjf zz
get cjf
dbsize
cluster info

中文乱码
chcp 65001


删除节点
ruby redis-trib.rb del-node <ip>:<port> 'node_id'   // 单引号内放置节点id
如果是删除从（Slave）节点，上述命令即可。
如果是删除主（Master）节点，则要看情况：
如果主节点上有从节点，则要将从节点删除或转移到其它主节点上去，该主节点才能被删除。
如果主节点上有槽（Slot），则要将槽删除或转移到其它主节点上去，该主节点才能被删除。
转移槽的方法：
redis-trib.rb reshard <ip>:<port>   // 取消分配的槽（Slot）的节点

How many slots do you want to move (from 1 to 16384)? <number>   // 填入的数字应该是该节点的全部槽，从 reshard 命令列出来的条目中得到
What is the receiving node ID? <node_id>   // 需要接收这些槽的主节点 id，就是那 40 位编码
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: <node_id>   // 要删除的主节点的 id
Source node #2: done   // 输入 done

Do you want to proceed with the proposed reshard plan (yes/no)? yes   // 输入 yes

```

