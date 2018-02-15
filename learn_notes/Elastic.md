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