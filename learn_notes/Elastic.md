# elasticsearch
```sh
挂载硬盘
azure ubuntu16.04有时候安装java9失败
sudo apt-get remove -y openjdk-9-jdk
sudo apt autoremove -y
sudo apt-get install -y openjdk-8-jdk

sudo vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH

source /etc/profile

如果报错
vm.max_map_count [65530] likely too low, increase to at least [262144]
sudo sysctl -w vm.max_map_count=262144

sudo apt install unzip -y
https://www.elastic.co/cn/downloads/elasticsearch

本地集群
./elasticsearch -Ehttp.port=9201 -Epath.data=node2
config是配置文件
elasticsearch.yml
cluster.name 集群名称
node.name 节点名称
network.host/http.port ip与端口，要填私有ip
path.data 数据存储地址
path.log 日志存储地址

http://ip:9200/_cat/nodes?v
```

# elasticsearch-head
```sh
sudo vi elasticsearch.yml 添加下面两句，冒号后面有空格
http.cors.enabled: true
http.cors.allow-origin: "*"

https://github.com/mobz/elasticsearch-head
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/
sudo apt-get install npm -y
sudo apt install nodejs-legacy -y
npm install
npm run start
http://ip:9100
```

# kibana
```sh
https://www.elastic.co/cn/downloads/kibana

tar -zxvf kibana.gz
sudo vi kibana.yml
server.host: "10.0.0.2"
server.port: "5601"
elasticsearch.url: "http://10.0.0.2:9200"

http://ip:5601

./kibana -e http://ip:9201 -p 8601

Dev Tools

POST /accounts/person/1
{
  "name":"zhangsan",
  "info":"haha"
}

GET /accounts/person/1

POST /accounts/person/1/_update
{
  "doc":{
  "name":"lisi",
  "info":"haha"
  }
}

DELETE /accounts/person/1

GET /accounts/person/_search?q=xxx   //全文检索

GET /accounts/person/_search
{
    "query":{
        "match":{
            "name":"zhangsan"
        }
    }
}
```

# Packetbeat
```sh
https://www.elastic.co/downloads/beats/packetbeat

sudo vi packetbeat.yml
sudo chown root:root packetbeat.yml
sudo ./packetbeat -e -c packetbeat.yml
```

# Logstash
```sh
https://www.elastic.co/cn/downloads/logstash
```

# [Logstash6.0.1同步MySQL到Elasticsearch6.0.1](https://www.pocketdigi.com/20171212/1585.html)
```sh
Logstash是一个数据收集管道，配置输入输出，可将数据从一个地方传到另一个地方。
默认Logstash不包含读取数据库的jdbc插件，需要手动下载。进入Logstash的bin目录，执行：
./logstash-plugin install logstash-input-jdbc
在某处建一个目录，放置配置文件(哪不重要，因为执行时会配置路径)，我这放到bin下的mysql目录里。
复制mysql jdbc驱动(mysql-connector-java-5.1.35.jar)到该目录下
编写导出数据的sql，放到sql.sql(文件名自己取)里,内容类似这样：
select *, id as car_id 
from violation_car car 
where car.gmt_modified>= :sql_last_value

这里需要介绍下，我的表里有个gmt_modified字段，用于记录该条记录的最后修改时间，sql_last_value是Logstash查询后，保存的上次查询时间，第一次查询时，该值是1970/1/1,所以第一次导入，如果你的表现有数据很多，可能会有点问题，后面会根据最后修改时间，更新修改过的数据。

编写输入输出配置文件jdbc.conf
input {
  jdbc {
    jdbc_driver_library => "/xxx/xxx/logstash-6.0.1/bin/mysql/mysql-connector-java-5.1.35.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/violation"
    jdbc_user => "root"
    jdbc_password => ""
    schedule => "* * * * *"
    jdbc_default_timezone => "Asia/Shanghai"
    statement_filepath => "/xxx/xxx/logstash-6.0.1/bin/mysql/sql.sql"
    use_column_value  => false
    last_run_metadata_path => "/xxx/xxx/logstash-6.0.1/bin/mysql/last_run.txt"
  }
}
output {
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "violation"
        document_id => "%{car_id}"
        document_type => "car"
    }
    stdout {
        codec => json_lines
    }
}

input.jdbc.jdbc_driver_library  jdbc驱动的位置
input.jdbc.jdbc_driver_class    驱动类名
input.jdbc.jdbc_connection_string   数据库连接字符串
input.jdbc.jdbc_user    用户名
input.jdbc.jdbc_password  密码
input.jdbc.schedule   更新计划(参考linux crontab)
input.jdbc.jdbc_default_timezone   时区，默认没有时区，日志里时间差8小时，中国需要用Asia/Shanghai
input.jdbc.statement_filepath 导出数据的sql文件，就是上面写的
input.jdbc.use_column_value  如果是true,sql_last_value是tracking_column指定字段的数字值，false就是时间，默认是false
input.jdbc.last_run_metadata_path  保存sql_last_value值文件的位置
output.elasticsearch.hosts elasticsearch服务器，填多个，请求会负载均衡。
output.elasticsearch.index 索引名
output.elasticsearch.document_id   生成文件的id,这里使用sql产生的car_id
output.elasticsearch.document_type  文档类型 
output.stdout 配置的是命令行输出，使用json

配置完以后，启动
./logstash -f mysql/jdbc.conf
按上面的配置，logstash会每分钟查询一次表，看是否会更新，有更新则提交到Elasticsearch

schedule参数说明
https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html
* 5 * 1-3 *
will execute every minute of 5am every day of January through March.
0 * * * *
will execute on the 0th minute of every hour every day.
0 6 * * * 
will execute at 6:00am every day.









### 当前版本logstash5.5.1
./bin/logstash-plugin install logstash-input-jdbc
wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.36/mysql-connector-java-5.1.36.jar
### vi jdbc-test.conf
input {
    stdin {
    }
    jdbc {
        # mysql jdbc connection string to our backup databse
        jdbc_connection_string => "jdbc:mysql://localhost:3306/test"
        # the user we wish to excute our statement as
        jdbc_user => "root"
        jdbc_password => ""
        # the path to our downloaded jdbc driver
        jdbc_driver_library => "path/to/mysql-connector-java-5.1.36.jar"
        # the name of the driver class for mysql
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_paging_enabled => "true"
        jdbc_page_size => "50000"
        ## 执行的sql 文件路径+名称 statement => "sql语句"
        statement_filepath => "./sql/jdbc.sql"
        ## schedule：设置监听间隔
        schedule => "* * * * *"
        type => "jdbc"
    }
}

filter {
    json {
        source => "message"
        remove_field => ["message"]
    }
}

output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "jdbc-test-%{+YYYY.MM.dd}"
    }
    stdout{          
        codec => rubydebug
    }
}

./bin/logstash -f ./jdbc-test.conf

```

# [Win10通过Logstash由MySQL和SQL Server向Elasticsearch导入数据](https://segmentfault.com/a/1190000011061797)
### [elasticsearch-6.0.0-beta2.msi](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.0-beta2.msi)
### [kibana-6.0.0-beta2-windows-x86_64.zip](https://artifacts.elastic.co/downloads/kibana/kibana-6.0.0-beta2-windows-x86_64.zip)
### [logstash-6.0.0-beta2.zip](https://artifacts.elastic.co/downloads/logstash/logstash-6.0.0-beta2.zip)
### Java-jdk-8u131-windows-x64.exe   
### [mysql-connector-java-5.1.44-bin.jar](https://dev.mysql.com/downloads/connector/j/)
### [mysql-installer-community-5.7.19.0.msi](https://dev.mysql.com/downloads/windows/installer/5.7.html)
### SQL Server 2016
### [Microsoft JDBC Driver 6.2 for SQL Server](https://www.microsoft.com/zh-CN/download/details.aspx?id=55539)
### 配置文件
```sh
我已在本地“hosts”（C:WindowsSystem32driversetchosts）文件中添加“127.0.0.1 es”，
以下内容中的“es”请自行替换为“localhost”

mysql.conf
input {
    jdbc {
        jdbc_driver_library => "C:/Elastic/logstash-6.0.0-beta2/lib/mysqldriver/mysql-connector-java-5.1.44-bin.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://es:3306/forelk?autoReconnect=true&useSSL=false"
        jdbc_user => "root"
        jdbc_password => "123456"
        schedule => "* * * * *"
        jdbc_default_timezone => "Asia/Shanghai"
		statement => "SELECT * FROM elktable;"
    }
}
output {
    elasticsearch {
        index => "elkdb"
        document_type => "elktable"
        document_id => "%{elkid}"
        hosts => ["es:9200"]
    }
}



RunKibana.vbs
Set ws = CreateObject("Wscript.Shell") 
ws.run "cmd /c kibana.bat",vbhide



sqlserver.conf
input {
    jdbc {
		jdbc_driver_library => "C:/Elastic/logstash-6.0.0-beta2/lib/sqlserverdriver/mssql-jdbc-6.2.1.jre8.jar"
        jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
		jdbc_connection_string => "jdbc:sqlserver://es:1433;databaseName=elkdb;"
		jdbc_user => "sa"
		jdbc_password => "123456"
        schedule => "* * * * *"
        jdbc_default_timezone => "Asia/Shanghai"
		statement => "SELECT * FROM elktable"
    }
}
output {
    elasticsearch {
        index => "sselkdb"
        document_type => "sselktable"
        document_id => "%{elkid}"
        hosts => ["es:9200"]
    }
}
```

# Windows安装elasticsearch-6.0.0-beta2.msi
```sh
注意安装路径，中间不要有中文和空格，这里我选择安装在C:\Elastic\Elasticsearch；
其他三个文件（lib、data、log）也放在C:\Elastic\Elasticsearch下（可通过勾选框一键修改）；
不安装其他插件，如x-pack（安装很慢，还需要配置以及license，后期可以加进去）；
在浏览器中访问“es:9200”

将kibana-6.0.0-beta2-windows-x86_64.zip和logstash-6.0.0-beta2.zip解压到C:\Elastic下

创建kibana的后台启动文件
可直接下载上方提供的配置文件，或自行在C:\Elastic\kibana-6.0.0-beta2-windows-x86_64\bin下创建RunKibana.vbs，编辑内容如下：
Set ws = CreateObject("Wscript.Shell")
   ws.run "cmd /c kibana.bat",vbhide

保存文件后双击“RunKibana.vbs”运行，然后在浏览器中访问“es:5601”。
正常跳出kibana的界面且没有错误提示说明kibana服务已正常启动
```

# 安装Java
```sh
JAVA_HOME : C:\Program Files\Java\jdk1.8.0_131（请根据自己的安装路径进行替换）；
CLASSPATH : .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\d
Path : %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
```

# 创建MySQL数据
```sh
创建数据库“forelk”，并在“forelk”下创建表“elktable”，之后插入数据：

CREATE DATABASE `forelk` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */;

CREATE TABLE `elktable` (
  `elkid` int(11) NOT NULL,
  `elkname` varchar(45) COLLATE utf8_bin DEFAULT NULL,
  `elkage` int(11) DEFAULT NULL,
  `elksex` tinyint(4) DEFAULT NULL,
  `elkbirth` date DEFAULT NULL,
  PRIMARY KEY (`elkid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

INSERT INTO `forelk`.`elktable`
(`elkid`,
`elkname`,
`elkage`,
`elksex`,
`elkbirth`)
VALUES
(111,
aa,
11,
1,
2006);

INSERT INTO `forelk`.`elktable`
(`elkid`,
`elkname`,
`elkage`,
`elksex`,
`elkbirth`)
VALUES
(222,
bb,
22,
0,
1995);
```

# 配置MySQL驱动
```sh
在C:\Elastic\logstash-6.0.0-beta2\lib下新建mysqldriver文件夹，并将mysql-connector-java-5.1.44-bin.jar拷贝到mysqldriver文件夹中。
```

# 准备Logstash的配置文件
```sh
在C:\Elastic\logstash-6.0.0-beta2\config下新建mysql.conf文件，编辑内容如下：
input {
    jdbc {
        jdbc_driver_library => "C:/Elastic/logstash-6.0.0-beta2/lib/mysqldriver/mysql-connector-java-5.1.44-bin.jar"
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_connection_string => "jdbc:mysql://es:3306/forelk?autoReconnect=true&useSSL=false"
        jdbc_user => "root"
        jdbc_password => "123qweASD"
        schedule => "* * * * *"
        jdbc_default_timezone => "Asia/Shanghai"
        statement => "SELECT * FROM elktable;"
    }
}
output {
    elasticsearch {
        index => "elkdb"
        document_type => "elktable"
        document_id => "%{elkid}"
        hosts => ["es:9200"]
    }
}

jdbc_driver_library：
数据库驱动路径，这里我填写的是绝对路径，可自行尝试相对路径；

jdbc_driver_class：
驱动名称；

jdbc_connection_string：
数据库的连接字符串；
forelk为数据库名；
?autoReconnect=true&useSSL=false自动重连并禁用SSL；

jdbc_user：
数据库用户名；

jdbc_password：
数据库密码；

schedule：
重复执行导入任务的时间间隔；

jdbc_default_timezone：
默认时区设置；

statement：
导入的表（查询SQL，可以过滤数据）

index:
索引名称（类似数据库名称）；

document_type：
类型名称（类似数据库表名）；

document_id：
类似主键；

hosts：
要导入到的Elasticsearch所在的主机;
```

# 导入MySQL
```sh
在C:\Elastic\logstash-6.0.0-beta2下Shift+鼠标右键，选择在此处打开命令窗口(W)；
执行bin\logstash -f config\mysql.conf
```

# 查询MySQL
```sh
在Kibana的Dev Tools下执行GET elkdb/_search命令，
右侧的结果显示有数据（我的MySQL表中插入了四条数据，所以这里导入的也是四条），
则说明导入成功。
```

# 创建SQL Server数据
```sh
在SQL Server 2016配置管理器中启用SQL Server网络配置下MSSQLSERVER的协议中的Named Pipes，然后重启SQL Server（MSSQLSERVER）服务。

USE [master]
GO
CREATE DATABASE [elkdb]

USE [elkdb]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE TABLE [dbo].[elktable](
    [elkid] [NVARCHAR](50) NOT NULL,
    [elkname] [NVARCHAR](50) NULL,
    [elkage] [INT] NULL,
    [elksex] [BIT] NULL,
    [elkbirth] [DATETIME] NULL
) ON [PRIMARY]
GO
ALTER TABLE [dbo].[elktable] ADD  CONSTRAINT [DF_elktable_elkid]  DEFAULT (NEWID()) FOR [elkid]
GO
ALTER TABLE [dbo].[elktable] ADD  CONSTRAINT [DF_elktable_elkbirth]  DEFAULT (GETDATE()) FOR [elkbirth]
GO

INSERT [dbo].[elktable] ([elkid], [elkname], [elkage], [elksex], [elkbirth]) VALUES (N'B0048C6E-05AD-4685-861F-E8F4A861D3AB', N'ssa', 123, 0, CAST(N'2017-09-08T09:39:12.567' AS DateTime))
GO
INSERT [dbo].[elktable] ([elkid], [elkname], [elkage], [elksex], [elkbirth]) VALUES (N'EAC8CC17-4E66-4C1C-964B-36D3CD875BDD', N'ssb', 456, 1, CAST(N'2017-09-08T09:39:23.090' AS DateTime))
GO
INSERT [dbo].[elktable] ([elkid], [elkname], [elkage], [elksex], [elkbirth]) VALUES (N'D672806D-6C4F-4CCB-9DCC-434A7ECDA083', N'ssc', NULL, 0, CAST(N'2017-09-08T09:39:33.050' AS DateTime))
GO
INSERT [dbo].[elktable] ([elkid], [elkname], [elkage], [elksex], [elkbirth]) VALUES (N'20DDA41B-A679-4627-B6ED-1D96AF953CF7', N'ssd', 789, NULL, CAST(N'2017-09-08T09:39:41.467' AS DateTime))
GO
INSERT [dbo].[elktable] ([elkid], [elkname], [elkage], [elksex], [elkbirth]) VALUES (N'6F9F8CD8-FCFB-4D88-B191-40DFD360C8C3', NULL, 135, NULL, CAST(N'2017-09-08T09:39:57.353' AS DateTime))
GO

```

# 准备SQL Server驱动
```sh
在C:/Elastic/logstash-6.0.0-beta2/lib下创建sqlserverdriver文件夹，并将mssql-jdbc-6.2.1.jre8.jar拷贝到该文件夹下。
```

# 准备SQL Server的配置文件
```sh
在C:/Elastic/logstash-6.0.0-beta2/config下创建sqlserver.conf
input {
    jdbc {
        jdbc_driver_library => "C:/Elastic/logstash-6.0.0-beta2/lib/sqlserverdriver/mssql-jdbc-6.2.1.jre8.jar"
        jdbc_driver_class => "com.microsoft.sqlserver.jdbc.SQLServerDriver"
        jdbc_connection_string => "jdbc:sqlserver://es:1433;databaseName=elkdb;"
        jdbc_user => "sa"
        jdbc_password => "123qweASD"
        schedule => "* * * * *"
        jdbc_default_timezone => "Asia/Shanghai"
        statement => "SELECT * FROM elktable"
    }
}
output {
    elasticsearch {
        index => "sselkdb"
        document_type => "sselktable"
        document_id => "%{elkid}"
        hosts => ["es:9200"]
    }
}
```

# 导入SQL Server
```sh
在C:\Elastic\logstash-6.0.0-beta2下Shift+鼠标右键，选择在此处打开命令窗口(W)；
执行bin\logstash -f config\sqlserver.conf
```

# 查询SQL Server
```sh
在Kibana的Dev Tools下执行GET sselkdb/_search命令
```
