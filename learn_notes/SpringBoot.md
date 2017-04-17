# 添加阿里云镜像
- /Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven2/conf/settings.xml
- /Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/conf/settings.xml
- /usr/local/Cellar/maven/3.3.9/libexec/conf/settings.xml
```sh
<mirror>
      <id>aliyun</id>
      <mirrorOf>*</mirrorOf>
      <name>aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
```
```sh
mvn install
会在target目录下新建一个jar包
```

- [maven数据库](maven repository)
- [spring data](http://projects.spring.io/spring-data/)