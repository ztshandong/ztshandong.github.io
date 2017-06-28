- [spring cloud docker](https://www.gitbook.com/book/eacdy/spring-cloud-book/details)
- [spring cloud](http://bbs.springcloud.cn/)
- [spring cloud](http://blog.didispace.com/Spring-Cloud%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)
- [spring cloud config](https://springcloud.cc/spring-cloud-config-zhcn.html)
- [spring cloud learn](https://springcloud.cc/)
- [ali diamond](http://jm.taobao.org/2012/04/17/diamond-1-intro/)
# 首先要安装maven
# server配置
# 一、pom.xml添加依赖
```java
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>1.2.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
            <version>1.2.2.RELEASE</version>
        </dependency>
```
# 二、启动类添加注解
- @EnableConfigServer
# 三、创建git项目，因为要用阿里云的服务器，所以用的阿里云code
```sh
1、创建项目，名为configserver，类型为public
如果改为private需要在第四步中填写alicode用户名和密码，但本人认证总是失败！！？？
2、创建文件conf-dev.yml，内容name: lisi
3、创建文件conf-test.yml，内容name: zhangsan
4、注意上面键值对中的：后面要用空格
curl username:password@localhost:8016/encrypt -d mysqlpassword
```
# 四、application.yml
```java
server:
  port: 8001

spring:
  cloud:
    config:
      server:
        git:
          uri: git@code.aliyun.com/ztshandong/configserver.git
  application:
    name: conf
# 使用ssh
```
# 五、访问server，git文件名是conf-dev，但是访问时要conf/dev
- http://serverip:8001/conf/dev/
- http://serverip:8001/conf/test/
- 返回格式，client中要用到name: "conf",profiles: ["dev"]
- 如果第三步中键值对中冒号后面没有空格，则propertySources的source是个document
```json
{
    name: "conf",
    profiles: [
    "dev"
    ],
    label: "master",
    version: "844ebb54e05085b72fcec8b750331dfae2f198ab",
    state: null,
    propertySources: [
    {
        name: "https://code.aliyun.com/ztshandong/configserver.git/conf-dev.yml",
        source: {
        name: "lisi"
        }
    }
    ]
}
```

# client配置
# 一、新建bootstrap.yml而不要用appliction.yml,因为bootstrap.yml会在应用启动之前读取
```java
对应config server中返回的conf和dev或test，本例git中文件名为conf-dev与conf-test
spring:
  application:
    name: conf
  cloud:
    config:
      uri:  http://configserverip:8001
      label: master
  profiles:
    active: test
management:
  security:
    enabled: false
---
server:
  port: 8002
```
# 二、pom.xml
```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
			<version>1.2.2.RELEASE</version>
		</dependency>
```
# 三、java
```java
启动类添加注解@RestController与@RefreshScope

    @Value("${name}")
    String myname;

    @RequestMapping("/name")
    public String showName() {
        return "The name is: " + myname;
    }
```
四、测试
```java
访问serverip
http://serverip:8001/conf/dev/
http://serverip:8001/conf/test/
访问clientip
http://clientip:8002/name
刷新配置，git文件的配置改变时通知程序刷新，可用webhook实现，management.security.enable=false
curl -X POST http://clientip:8002/refresh

```

