# [docker gitbook](https://www.gitbook.com/book/yeasy/docker_practice/details)
# [aliyun docker](https://dev.aliyun.com/search.html)
# [hub docker](https://hub.docker.com/explore/)
# [163](https://c.163.com/hub#/m/home/)
# [阿里云搭建](https://yq.aliyun.com/articles/57265?spm=5176.100239.blogcont57276.11.Del2Z4)
# [搭建docker](http://morning.work/page/2016-01/deploying-your-own-private-docker-registry.html)
- 阿里云-容器服务-镜像-镜像仓库控制台  

# 目前用ubuntu运行docker最好，所以服务器最好用ubuntu系统
# docker运行在linux内核下使用真机ip即可访问，mac和windows都不行，都需要用虚拟机的ip才能访问
# MAC
```sh
反正mac运行docker不怎么样
创建docker虚拟机，首先下载DockerToolbox，10.10.3以上的用docker-for-mac
http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/
装完后记得配置加速镜像

docker-for-mac安装过就不用执行下面的了
brew install -y docker   
sudo docker-machine rm default
docker-machine create --engine-registry-mirror=https://0rnhdnox.mirror.aliyuncs.com --engine-insecure-registry=192.168.125.152：2375 -d virtualbox default
上句命令会从github上下载最新的boot2docker，会很慢，你懂的，将提前下载好的boot2docker.iso复制到提示的缓存路径下/Users/zhangtao/.docker/machine/cache/
docker-machine env default
/Users/zhangtao/.docker/machine/machines/default/config.json
eval "$(docker-machine env default)"
docker info
```
# Docker远程，此方法失败，这个模式好像还要配证书，否则SSL错误，暂时不搞
```sh
CentOS安装Docker不建议使用yum install docker   假如ip=192.168.125.152
使用 curl -fsSL https://get.docker.com/ | sh
或   yum install docker-engine
docker version 验证
开启docker远程API
vi /usr/lib/systemd/system/docker.service
ExecStart这一行后面加上-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock 
systemctl daemon-reload
systemctl restart docker
netstat -anp | grep 2375

vi ~/.bash_profile
export DOCKER_HOST=tcp://192.168.125.152:2375
source ~/.bash_profile
mvn clean package docker:build -DskipTests   这样就是Docker服务器build了
添加insecure-registry=192.168.125.152:2375
curl 192.168.125.152:2375/info   成功
docker -H tcp://192.168.125.152:2375 info  这样会提示失败，关于SSL的错误信息，应该是证书没有配置
```
# maven pom.xmd
```java
            <plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
						<configuration>
							<outputDirectory>${project.basedir}/docker</outputDirectory>
							<!--&lt;!&ndash;非必填项，即在生成的jar包名称后面追加该分类名称&ndash;&gt;-->
							<!--<classifier>boot</classifier>-->
							<!--<mainClass>cn.qlt.service.UserService</mainClass>-->
						</configuration>
					</execution>
				</executions>
			</plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <imageName>mydocker</imageName>
                    <!--<dockerDirectory>${project.basedir}/docker</dockerDirectory>-->
                    <!--上一条命令使用Dockerfile-->
                    <baseImage>java</baseImage>
                    <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>/${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
```
# Dockerfile，这种模式要将生成的jar包上传到git，pom.xml设置生成的jar包路径为${project.basedir}/docker
```java
FROM java:8
MAINTAINER zhangtao  
ADD docker/demo-0.0.1-SNAPSHOT.jar /demo-0.0.1-SNAPSHOT.jar
CMD ["java","-jar","demo-0.0.1-SNAPSHOT.jar"]
```
# 生成
```sh
mvn package  即可在${project.basedir}/docker下生成一个jar包，源码上传后阿里云会自动生成一个镜像
mvn package docker:build  这样一并生成docker image，本机调试使用
前两个是用maven生成
docker build -t="dockername" .

如果报错
pkill docker
iptables -t nat -F
ifconfig docker0 down
brctl delbr docker0
systemctl restart docker
```
# 常用命令
```sh
将docker里的文件拷出来
docker cp <Container ID>:<Container path> <host path>
将主机里的文件拷进去
docker cp <Host path> <Container ID>:<Container path>
docker ps -a   显示所有container
docker images
docker rm <containerid>
docker rmi <imageid>

# 杀死所有container
docker kill $(docker ps -a -q)
# 删除所有已经停止container
docker rm $(docker ps -a -q)
# 删除所有未打标签镜像
docker rmi $(docker images -q -f dangling=true)
# 删除所有镜像
docker rmi $(docker images -q)

docker ps
docker exec -it containerid bash  进入容器

docker run ubuntu apt-get update
# 上面命令会生成一个container，先获得其 id
# -l：显示最新的 container
docker ps -l
# 提交改动，并覆盖原有的 Ubuntu 镜像
docker commit <container_id> ubuntu
# 这样之前的ubuntu镜像Tag就会变为null，删除时会报错，查出IMAGE ID后添加TAG
docker tag 6a2f32de169d ubuntu:del
docker rmi ubuntu:del

docker run -it centos  这样就会自动进入到docker里
# 这样的镜像如果直接exit就会恢复到原始状态，要开启另一个终端，然后commit

# 下面三句看看吧
- export MACHINE_DOCKER_INSTALL_URL=http://docker-mirror.oss-cn-hangzhou.aliyuncs.com
- --engine-insecur-registry registry.mirrors.aliyuncs.com
- docker build . --build-arg APT_MIRROR="http://mirrors.163.com"
vi /etc/docker/daemon.json
```
# Node ReactNative
```sh
docker pull centos
yum -y install npm
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
npm install -g cnpm --registry=https://registry.npm.taobao.org
yum -y install curl 
cnpm install -g n
n stable
npm install --global vue-cli browserify gulp babel

npm install -g yarn react-native-cli
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```
# MySQL
```sh
docker pull mysql:5.7
第一步只是为了建立自己的数据库和用户，这个镜像运行一下稍微等一会再停掉就可以，不要马上停止，因为可能还没创建好数据库
docker run -p 3306:3306 -v /data/mysql57-1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_DATABASE=testDB1 -e MYSQL_USER=testuser1 -e MYSQL_PASSWORD=12345678 -d mysql:5.7
第二步才是真正要用的镜像，起个名字并且开机启动
docker run -p 3306:3306 --name=mysql5.7-1 -v /data/mysql57-1:/var/lib/mysql --restart=always -d mysql:5.7
然后删除第一步的container

创建多个就很方便了
docker run -p 3306:3306 -v /data/mysql57-2:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_DATABASE=testDB2 -e MYSQL_USER=testuser2 -e MYSQL_PASSWORD=12345678 -d mysql:5.7

docker run -p 3306:3306 --name=mysql5.7-2 -v /data/mysql57-2:/var/lib/mysql --restart=always -d mysql:5.7

```
# MySQL主从复制
```sh
创建my-master.cnf
[client]
port        = 3306
socket      = /var/run/mysqld/mysqld.sock

[mysqld_safe]
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
nice        = 0

[mysqld]
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
port        = 3306
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp

log-bin = mysql-bin 
server-id = 1 
#auto-increment-increment = 2
#auto-increment-offset = 1
#binlog-do-db=dbTest 
#replicate-do-db=dbTest 

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address   = 127.0.0.1

#log-error  = /var/log/mysql/error.log

# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
!includedir /etc/mysql/conf.d/

创建my-slave.cnf
[client]
port        = 3306
socket      = /var/run/mysqld/mysqld.sock

[mysqld_safe]
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
nice        = 0

[mysqld]
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
port        = 3306
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /tmp
lc-messages-dir = /usr/share/mysql
explicit_defaults_for_timestamp

log-bin = mysql-bin 
server-id = 2
#auto-increment-increment = 2
#auto-increment-offset = 2
#binlog-do-db=dbTest 
#replicate-do-db=dbTest 

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address   = 127.0.0.1
#log-error  = /var/log/mysql/error.log
# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
!includedir /etc/mysql/conf.d/


启动主库
docker run -p 3307:3306 -v /data/mysql57-master:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678  -d mysql:5.7
docker run -p 3307:3306 --name=mysql57-master -v /data/mysql57-master:/var/lib/mysql -v ~/my-master.cnf:/etc/mysql/my.cnf --restart=always -d mysql:5.7
启动从库
docker run -p 3308:3306 -v /data/mysql57-slave:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678  -d mysql:5.7
docker run -p 3308:3306 --name=mysql57-slave -v /data/mysql57-slave:/var/lib/mysql -v ~/my-slave.cnf:/etc/mysql/my.cnf --restart=always -d mysql:5.7

主库
GRANT REPLICATION SLAVE ON *.* TO 'repuser'@'%' IDENTIFIED BY '12345678';
flush privileges; 
flush tables with read lock;
show master status;
此例中File是mysql-bin.000001， Position是2602
navicat同步数据
unlock tables;

从库
若是操作了从库可能导致同步失败，可先将从库的停止，清除，然后重新同步
stop slave;
reset slave; 

docker inspect --format '{{ .NetworkSettings.IPAddress }}' mysql57-slave
ip addr show docker0 终端中运行，不是数据库，本人是172.17.0.1，就是docker的网关

这种方法要自己手动建好数据库并且同步完数据后进行
change master to master_host='172.17.0.1',master_user='repuser',master_password='12345678',
master_log_file='mysql-bin.000001',master_log_pos=2602,master_port=3307;

如果报错就把slave中自定义数据库删掉，master_log_pos=4，这样slave会自动创建数据库并同步数据
change master to master_host='172.17.0.1',master_user='repuser',master_password='12345678',
master_log_file='mysql-bin.000001',master_log_pos=4,master_port=3307;
start slave;
show slave status;
```
# Redis
```sh
docker pull redis:3
docker run -d -p 6378:6379 --name=redis1 redis:3
docker exec -it redis1 redis-cli
config set requirepass 12345678
auth 12345678
config get requirepass

用另一个redis连接上面的redis
docker exec -it redis1 redis-cli -h 111.111.111.111 -p 6378 -a 12345678
```
# MySql Cluster
```sh
docker pull mysql/mysql-cluster:7.5
docker network create cluster --subnet=192.168.0.0/16
docker run -d --net=cluster -p 8587:3306 --name=management1 --ip=192.168.0.2 mysql/mysql-cluster:7.5 ndb_mgmd
docker run -d --net=cluster --name=ndb1 --ip=192.168.0.3 mysql/mysql-cluster:7.5 ndbd
docker run -d --net=cluster --name=ndb2 --ip=192.168.0.4 mysql/mysql-cluster:7.5 ndbd
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 mysql/mysql-cluster:7.5 mysqld
docker logs mysql1 2>&1 | grep password
docker exec -it mysql1 mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345678';

docker run -d --net=cluster --name=mysql2 --ip=192.168.0.11 mysql/mysql-cluster:7.5 mysqld
docker logs mysql2 2>&1 | grep password
docker exec -it mysql2 mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345678';

use mysql;
select user,host from user where user='root';
update user set host='%' where user = 'root';

docker run -it --net=cluster mysql/mysql-cluster:7.5
```
# RabbitMQ
```sh
docker pull rabbitmq:3-management
docker run -d -p 15672:15672 -p 5672:5672  --name rabbit3 -e RABBITMQ_DEFAULT_USER=username -e RABBITMQ_DEFAULT_PASS=password rabbitmq:3-management
http://ip:15672  即可访问
```

# docker安装eclipse,tomcat,jdk,这样可以确保和服务器的环境是一样
# centos下亲测可用，mac还不行，windows未测
```java
centos删除自带jdk，安装GNOME的时候又会装上openjdk，不过没关系
rpm -e --nodeps `rpm -qa | grep java`
安装官方jdk
rpm -ivh jdk-7u79-linux-x64.rpm
查看安装路径,rpm安装方式默认会把jdk安装到/usr/java/jdk1.7.0_79，然后通过三层链接，链接到/usr/bin
find / -name java
vi /etc/profile
export PATH=".;$PATH:$JAVA_HOME/bin"
export JAVA_HOME=/usr/java/jdk1.7.0_79
export CLASS_PATH="$JAVA_HOME/lib"

source /etc/profile 

查看链接
cd /bin 
ll | grep java

tomcat
这样可以做tomcat集群
mkdir /wocloud/tomcat_cluster/
mkdir /wocloud/tomcat_cluster/tomcat1
mv /download/apache-tomcat-7.0.77/ /wocloud/tomcat_cluster/tomcat1/
配置一下tomcat的环境变量和内存设置，vi /wocloud/tomcat_cluster/tomcat1/apache-tomcat-7.0.77/bin/catalina.sh文件，并在其中加入如下配置：
JAVA_OPTS="-Xms512m -Xmx1024m -Xss1024K -XX:PermSize=512m -XX:MaxPermSize=1024m"
export TOMCAT_HOME=/wocloud/tomcat_cluster/tomcat1/apache-tomcat-7.0.77
export CATALINA_HOME=/wocloud/tomcat_cluster/tomcat1/apache-tomcat-7.0.77
export JRE_HOME=/usr/java/jdk1.7.0_79/jre
export JAVA_HOME=/usr/java/jdk1.7.0_79

cd /wocloud/tomcat_cluster/tomcat1/apache-tomcat-7.0.77/bin/
./startup.sh

解压eclipse
yum -y install gtk2.i686 gtk2-engines.i686 PackageKit-gtk-module.i686 PackageKit-gtk-module.x86_64 libcanberra-gtk2.x86_64 libcanberra-gtk2.i686 ghostscript-chinese-zh_CN.noarch kde-l10n-Chinese，glibc-common

vim /etc/profile
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL LC_ALL
export LC_ALL=en_US.UTF-8

真linux内核不装GNOME也可以，MAC装了也不行
yum -y groups install "GNOME Desktop"

最后加bash是为了防止关闭eclipse之后会退出容器
可以给容器起个名字，参照导入导出

docker run  -it -p 8080:8080 -v ~/eclipsepro:/eclipsepro -v ~/Downloads:/Downloads -v /tmp/.X11-unix:/tmp/.X11-unix --net=host -e DISPLAY=$DISPLAY eclipse7 env LANG=en_US.UTF-8 sh -c 'cd /eclipse&&/eclipse/./eclipse&&bash'

如果修改了eclipse的配置就要docker commit了
```
# 导入导出，建议用save，也大不了多少
```sh
docker save [ImageID] > eclipsex.tar
docker load < eclipsex.tar
load之后名字和版本都是null，所以要tag一下
docker tag [ImageId] eclipse:marsjdk7
docker run --name=eclipse7 -it -p 8080:8080 -v ~/eclipsepro:/eclipsepro -v ~/Downloads:/Downloads -v /tmp/.X11-unix:/tmp/.X11-unix --net=host -e DISPLAY=$DISPLAY eclipse:marsjdk7 env LANG=en_US.UTF-8 sh -c 'cd /eclipse&&/eclipse/./eclipse&&bash'    容器起个名字
docker start eclipse7    不带-it参数的话关闭eclipse就会退出容器
docker stop eclipse7
docker rm eclipse7
```
# 使用阿里镜像，体积会压缩到大概一半左右的样子
```sh
sudo docker pull registry.cn-hangzhou.aliyuncs.com/ztshandong/eclipsex:170518      

docker run --name=alieclipse -it -p 8080:8080 -v ~/eclipsepro:/eclipsepro -v ~/Downloads:/Downloads -v /tmp/.X11-unix:/tmp/.X11-unix --net=host -e DISPLAY=$DISPLAY registry.cn-hangzhou.aliyuncs.com/ztshandong/eclipsex:170518  env LANG=en_US.UTF-8 sh -c 'cd /eclipse&&/eclipse/./eclipse&&bash'
docker start alieclipse    
docker stop alieclipse
docker rm alieclipse
```

