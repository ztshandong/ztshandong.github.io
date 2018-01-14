# Ubuntu
```sh
sudo useradd -m hadoop -s /bin/bash
sudo passwd hadoop
sudo adduser hadoop sudo

配置ssh，略

apt-cache search jdk
sudo apt-get install -y openjdk-8-jdk

设置JAVA_HOME
在Ubuntu中有如下几个文件可以设置环境变量
1、/etc/profile:在登录时,操作系统定制用户环境时使用的第一个文件,此 文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。
2、/etc/environment:在登录时操作系统使用的第二个文件,系统在 读取你自己的profile前,设置环境文件的环境变量。
3、~/.bash_profile:在登录时用到的第三个文件是.profile文 件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用 户登录时,该 文件仅仅执行一次!默认情况下,他设置一些环境变游戏量,执 行用户的.bashrc文件。/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该 文件被读取.
4、~/.bashrc:该文件包含专用于你的bash shell的bash信 息,当登录时以及每次打开新的shell时,该该文件被读取。

sudo vi /etc/environment
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib 
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:$JAVA_HOME/bin"
source /etc/environment

貌似设置了environment就可以了
sudo vi /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
source /etc/profile

wget https://mirrors.cnnic.cn/apache/hadoop/common/hadoop-3.0.0/hadoop-3.0.0.tar.gz
sudo tar -zxvf hadoop-3.0.0.tar.gz -C ~
sudo mv ./hadoop-3.0.0 ./hadoop
sudo chown -R hadoop:hadoop ./hadoop
cd ~/hadoop
./bin/hadoop version

默认为单机模式，节点既作为NameNode也作为DataNode
伪分布模式需要修改core-site.xml与hdfs-site.xml与mapred-site.xml，位于~/hadoop/etc/hadoop
sudo vi core-site.xml
<configuration>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/home/root/hadoop/tmp</value>
                <description>Description</description>
        </property>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://localhost:9000</value>
        </property>
</configuration>
tmp是设置临时数据，否则每次启动都要执行hadoop namenode -formate
fs.defaultFS为访问名称

sudo vi hdfs-site.xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/home/root/hadoop/tmp/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/home/root/hadoop/tmp/dfs/data</value>
        </property>
</configuration>
dfs.replication表示副本数量，伪分布要设置1
dfs.namenode.name.dir表示本地磁盘目录，存储fsimage文件的地方
dfs.datanode.data.dir表示本地磁盘目录，HDFS数据存放block的地方

初始化hadoop namenode -formate
启动所有进程start-all.sh

命令
hadoop fs适用于任何系统，本机模式只能用这个访问
hadoop dfs与hdfs dfs只适用于hdfs文件系统

Hadoop集群包含
1.NameNode：负责协调集群中的数据存储，就是存储一个目录，应用首先访问NameNode，询问数据存储的位置。NameNode会告诉应用这个数据被分成了几块，每块的存储位置，NameNode中的元数据信息存储在RAM中。
2.DataNode：存储被拆分的数据块
MapReduce包含3和4
3.JobTracker：协调数据计算任务，MapReduce作业会拆分为多个小作业执行，由JobTracker协调
4.TaskTracker：不同的机器上部署TaskTracker，负责执行由JobTracker指派的任务
5.SecondaryNameNode：帮助NameNode收集文件系统运行的状态信息，是HDFS的一个组件，不是热备，是冷备份，为了加速启动
```