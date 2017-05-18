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
docker ps -a
docker images
docker rm <containerid>
docker rmi <imageid>

# 杀死所有容器
docker kill $(docker ps -a -q)
# 删除所有已经停止container
docker rm $(docker ps -a -q)
# 删除所有未打标签镜像
docker rmi $(docker images -q -f dangling=true)
# 删除所有镜像
docker rmi $(docker images -q)

docker ps
docker exec -it containerid bash  进入镜像

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
# MySQL
```sh
docker pull mysql
docker run -p 3306:3306 --name mysql -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=19820108 -eMYSQL_DATABASE=testDB -e MYSQL_USER=testuser -e MYSQL_PASSWORD=12345678 --restart=always -d mysql
```

# docker安装eclipse,tomcat,jdk,这样可以确保和服务器的环境是一样,这个还是有点价值的
```java
centos删除自带jdk，后面不知道哪一步又装上了，可能是装eclipse依赖的那些，不过没关系
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
考虑后面要做tomcat集群，所以建立新目录并将解压好的tomcat移进去：
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

安装eclipse
启动eclipse，图形化总是失败，后来发现加上--net=host就可以，GNOME没有必要安装，最后加bash是为了防止关闭eclipse之后会退出镜像
docker run -it -p 8080:8080 -v /eclipsepro:/eclipsepro -v ~/Downloads:/download --net=host eclipse  sh -c 'cd /eclipse&&/eclipse/./eclipse&&bash'
但是要记得，如果修改了eclipse的配置就要docker commit了
--net=host之后就不用再加-e DISPLAY，也不用装下面这一堆东西

yum install gtk2.i686 gtk2-engines.i686 PackageKit-gtk-module.i686 PackageKit-gtk-module.x86_64 libcanberra-gtk2.x86_64 libcanberra-gtk2.i686
yum groups install "GNOME Desktop"
```


