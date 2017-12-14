# 安装
```sh
不要装tomcat，好像会冲突
java 1.8
docker pull mysql:5.6
docker run -p 3306:3306 -v ~/DB/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_USER=jira -e MYSQL_PASSWORD=12345678 -d mysql:5.6

docker run -p 3306:3306 --name=mysql5.6jira -v ~/DB/mysql:/var/lib/mysql --restart=always -d mysql:5.6

wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-7.4.1-x64.bin
chmod a+x atlassian-jira-software-7.4.1-x64.bin
sudo ./atlassian-jira-software-7.4.1-x64.bin

关闭已启动的jira，然后把破解包里面的atlassian-extras-3.2.jar和mysql-connector-java-5.1.42-bin.jar两个文件复制到/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/目录下
sudo cp atlassian-extras-3.2.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
sudo cp mysql-connector-java-5.1.42-bin.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/

sudo /opt/atlassian/jira/bin/stop-jira.sh 
sudo /opt/atlassian/jira/bin/./start-jira.sh 

http://ip:8080
连接mysql数据库要指定utf8
jira?autoReconnect=true&useUnicode=true&characterEncoding=UTF8 
```
