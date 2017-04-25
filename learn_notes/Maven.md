# 安装
- yum -y install maven
- brew install -y maven
- mvn -v
- /usr/share/maven/conf/settings.xml
- /Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven2/conf/settings.xml
- /Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/conf/settings.xml

# 加速
```sh
    <mirror>
      <id>aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    <mirror>  
        <id>osc</id>  
        <mirrorOf>*</mirrorOf>
        <name>oschina</name>
        <url>http://maven.oschina.net/content/groups/public/</url>  
    </mirror>  
```
- /src/main/resources/application使用.yml扩展名，否则中文有乱码

```sh
部署方法之一，在项目路径下
mvn install 或者 mvn package
会在target目录下新建一个jar包
```