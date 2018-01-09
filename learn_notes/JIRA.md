# 安装
```sh
java 1.8
sudo apt-get update
sudo apt-get install openjdk-8-jdk -y --fix-missing
docker pull mysql:5.6
docker run -p 3306:3306 -v ~/DB/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -e MYSQL_USER=jira -e MYSQL_PASSWORD=12345678 -d mysql:5.6

docker run -p 3306:3306 --name=mysql5.6jira -v ~/DB/mysql:/var/lib/mysql --restart=always -d mysql:5.6

wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-7.4.1-x64.bin
chmod a+x atlassian-jira-software-7.4.1-x64.bin
sudo ./atlassian-jira-software-7.4.1-x64.bin

关闭已启动的jira，然后把破解包里面的atlassian-extras-3.2.jar和mysql-connector-java-5.1.42-bin.jar两个文件复制到/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/目录下
sudo cp atlassian-extras-3.2.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
sudo cp mysql-connector-java-5.1.42-bin.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/

sudo /opt/atlassian/jira/bin/./stop-jira.sh 
sudo /opt/atlassian/jira/bin/./start-jira.sh 

http://ip:8080
连接mysql数据库要指定utf8
jira?autoReconnect=true&useUnicode=true&characterEncoding=UTF8 
```
# confluence
```sh
创建mysql数据库及用户
wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-5.6.6-x64.bin

wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-5.4.4-x64.bin

chmod 755 atlassian-confluence-5.6.6-x64.bin

sudo ./atlassian-confluence-5.6.6-x64.bin

打开浏览器 输入http://服务器ip:8090，记下Server IDservice confluence stop#停掉Confluence 服务
将confluence5.1-crack.zip 解压
将/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-2.4.jar 复制出来。替换confluence5.1-crack中的atlassian-extras-2.4.jar

chmod +x keygen.sh
./keygen.sh 
执行破解文件
注：必须是在图形界面下，因为这个运行需要图形。如果没有图形，那么就会报错。【1】输了Name，及之前记录下来的Server ID，按.patch! 选择需要破解的atlassian-extras-2.4.jar
破解后拷回去

ip:8090记录serverID  BQ6R-I4BW-ML9V-Z71E
AAABMQ0ODAoPeJxtkMtuwjAQRff+CktdB5EESkGyVBN7ETUPSgJVuzPuAJaCg/yISr++gZRN1eXM3
Dk6Mw+1B8xA4nCOw+liEi/iJ5xUNY7G4QwxsNKos1OtJkmr940HLQEV/rQDU+43FowlQYgSA+IaY
sIBuW4GYRSEc9TvOCFdIU5Avo9CH5xokexBo76rOiDOeLineC5UQ5TulFW7Bp6tBA0j3SDeicbf+
GQvGgsDIVP93EJ9OcONn5R5ztdJSjPUg7QDLXpX/nVW5jJ4xfHNK5oOgPsVSeOtA1O0n2DJGFW8I
O/lBuf0heOcY4oryvCKFoyOUGkOQis7yKhiqyq1zDiuOc1RBaYDkzKyfH1cB+lk+Rbk2XwbfMxCj
n5t+2mWsnv1v9zKG3kUFv488wcoVonuMC4CFQCPPgQKVvni7h6KYfWHrYOUmw/e+gIVAJIWJhc31
y8tSnDeMY81cACudQ4oX02fb


sudo /etc/init.d/confluence stop
sudo cp atlassian-extras-2.4.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
sudo cp Confluence-5.4.4-language-pack-zh_CN.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
sudo cp mysql-connector-java-5.1.32-bin.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
sudo /etc/init.d/confluence start


sudo /etc/init.d/confluence stop
cd /opt/atlassian/confluence/confluence/WEB-INF/lib
ll |grep atlassian-extra | wc –l
sudo rm -fr atlassian-extra*

解压破解包，然后把里面的atlassian-extras-3.2.jar、Confluence-5.6.6-language-pack-zh_CN.jar、mysql-connector-java-5.1.39-bin.jar三个jar文件复制到/opt/atlassian/confluence/confluence/WEB-INF/lib目录下

sudo /etc/init.d/confluence start

java -jar confluence_keygen.jar


```