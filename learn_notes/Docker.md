# [docker gitbook](https://www.gitbook.com/book/yeasy/docker_practice/details)
# [aliyun docker](https://dev.aliyun.com/search.html)
# [hub docker](https://hub.docker.com/explore/)
# [163](https://c.163.com/hub#/m/home/)
# [阿里云搭建](https://yq.aliyun.com/articles/57265?spm=5176.100239.blogcont57276.11.Del2Z4)
# [搭建docker](http://morning.work/page/2016-01/deploying-your-own-private-docker-registry.html)
- 阿里云-容器服务-镜像-镜像仓库控制台  

- export MACHINE_DOCKER_INSTALL_URL=http://docker-mirror.oss-cn-hangzhou.aliyuncs.com
- --engine-insecur-registry registry.mirrors.aliyuncs.com
- docker build . --build-arg APT_MIRROR="http://mirrors.163.com"
# CentOS作为Docker服务器
```sh
安装Docker不建议使用yum install docker
使用 curl -fsSL https://get.docker.com/ | sh
或   yum install docker-engine
docker version 验证
开启docker远程API
vi /usr/lib/systemd/system/docker.service
ExecStart这一行后面加上-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H fd://加速镜像
systemctl daemon-reload
systemctl restart docker
netstat -anp | grep 2375
curl 127.0.0.1:2375/info

客户端将DOCKER_HOST 设置为tcp://dockerserverip:port
mvn clean package docker:build -DskipTests   这样就是Docker服务器build了
```
# maven
```java
 <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <imageName>mydocker</imageName>
                    <!--<dockerDirectory>${project.basedir}/docker</dockerDirectory>-->
                    <!--上一条命令使用Dockerfile-->
                    <baseImage>java</baseImage>
                    <baseImage>tomcat</baseImage>
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


mvn package docker:build
docker run -p 8080:8080 mydocker
```
# MAC
```sh
安装DockerToolbox，运行命令行模式，下载很慢，将boot2docker.iso复制到提示的缓存路径下
加速
docker-machine rm default
docker-machine create --engine-registry-mirror=https://wgaccbzr.mirror.aliyuncs.com -d virtualbox default
docker-machine env default
eval "$(docker-machine env default)"
docker info

docker build -t="dockername" .
# 杀死所有容器
docker kill $(docker ps -a -q)
# 删除所有已经停止容器
docker rm $(docker ps -a -q)
# 删除所有未打标签镜像
docker rmi $(docker images -q -f dangling=true)
# 删除所有镜像
docker rmi $(docker images -q)
```  
# 常用命令
```sh
docker exec -it alisql bash
docker images
docker ps -a
docker run ubuntu apt-get update
# 上面命令会生成一个container，先获得其 id
# -l：显示最新的 container
docker ps -l
# 提交改动，并覆盖原有的 Ubuntu 镜像
docker commit <container_id> ubuntu
# 这样之前的ubuntu镜像Tag就会变为null，删除时会报错，查出IMAGE ID后添加TAG
docker tag 6a2f32de169d ubuntu:del
docker rmi ubuntu:del
```
# CentOS
```sh
docker pull centos
docker run -it centos  这样就会自动进入到docker里
# 然后参照CentOS7的教程把镜像改成阿里云，如果需要安装develop-tools参照AliSQL
# 这样的镜像如果直接exit就会恢复到原始状态，要开启另一个终端，然后commit
```
# MySQL
```sh
docker pull mysql
docker run -p 3306:3306 --name mysql -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=19820108 -eMYSQL_DATABASE=testDB -e MYSQL_USER=testuser -e MYSQL_PASSWORD=12345678 --restart=always -d mysql
```






