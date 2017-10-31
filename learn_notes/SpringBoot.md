# application.yml  
```java
Spring Boot ::        (v1.5.4.RELEASE)

spring:
  datasource:
    url: jdbc:sqlserver://localhost;databaseName=dbname
    username: sa
    password: 12345678@Abc
    driverClassName: com.microsoft.sqlserver.jdbc.SQLServerDriver
  jpa:
    show-sql: true
    hibernate:
      dialect: org.hibernate.dialect.SQLServer2012Dialect
      naming:
        strategy: com.firs.ImprovedNamingStrategyEx    //修改命名策略，这个1.5.4无效
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl //用这个就够了
```
```java
package com.firs;

import org.hibernate.cfg.ImprovedNamingStrategy;
import org.hibernate.cfg.NamingStrategy;
import org.hibernate.internal.util.StringHelper;

import java.io.Serializable;
import java.util.Locale;

/**
 * Created by zhangtao on 2017/6/11.
 */
public class ImprovedNamingStrategyEx extends ImprovedNamingStrategy {
   private static final long serialVersionUID = 1L;
    public static final ImprovedNamingStrategyEx INSTANCE = new ImprovedNamingStrategyEx();
    @Override
    public String columnName(String columnName) {
        return columnName;
    }

    @Override
    public String tableName(String tableName) {
        return "test";
    }
    @Override
    public String classToTableName(String className) {
        return className;
    }
    @Override
    public String propertyToColumnName(String propertyName) {
        return propertyName;
    }

    @Override
    public String collectionTableName(
            String ownerEntity, String ownerEntityTable, String associatedEntity, String associatedEntityTable,
            String propertyName
    ) {
        return tableName( ownerEntityTable + '_' + propertyToColumnName(propertyName) );
    }

    protected static String addUnderscores(String name) {
        StringBuilder buf = new StringBuilder( name.replace('.', '_') );
        for (int i=1; i<buf.length()-1; i++) {
            if (
                    Character.isLowerCase( buf.charAt(i-1) ) &&
                            Character.isUpperCase( buf.charAt(i) ) &&
                            Character.isLowerCase( buf.charAt(i+1) )
                    ) {
                buf.insert(i++, '_');
            }
        }
        return buf.toString().toLowerCase(Locale.ROOT);
    }
}

```
# 自定义异常
```java
//一、创建错误信息类
public class ErrorInfo<T> {
    public static final Integer OK = 0;
    public static final Integer ERROR = 100;
    private Integer code;
    private String message;
    private String url;
    private T data;
}
//二、创建自定义异常，捕获异常
public class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }
}
//三、处理异常
import org.apache.catalina.servlet4preview.http.HttpServletRequest;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = MyException.class)
    @ResponseBody
    public ErrorInfo<String> jsonErrorHandler(HttpServletRequest req, MyException e) throws Exception {
        ErrorInfo<String> r = new ErrorInfo<>();
        r.setMessage(e.getMessage());
        r.setCode(ErrorInfo.ERROR);
        r.setData("Some Data");
        r.setUrl(req.getRequestURL().toString());
        return r;
    }
}
//四、Controller增加映射
@Controller
public class HelloController {
    @RequestMapping("/json")
    public String json() throws MyException {
        throw new MyException("发生错误2");
    }
}
```
# 独立部署
```java
http://blog.csdn.net/asdfsfsdgdfgh/article/details/52127562
SpringBoot的应用可以直接打成一个可运行的jar包， 
你无需发愁为了不同应用要部署多个Tomcat。但是实际部署时你会发现打成Jar包的方式有一个致命的缺点， 
当你改动了一个资源文件、或者一个类时， 打要往服务器重新上传全量jar包。
这样你本地开发环境直接用控制台方式运行，部署到服务器时打成普通war包部署。这样既享受到了SpringBoot开发带来的快感， 
又避免了增量部署不方便的问题。可谓两全其美。

修改pom.xml将打包方式改成war
<packaging>war</packaging>

SpringBoot默认Servlet容器是基于Tomcat8的
要支持低版本Tomcat需要在maven中指定Tomat版本，配置如下：
<properties>
    <tomcat.version>7.0.69</tomcat.version>
</properties>
然后依赖中加上（这个其实不加也行， 官方文档是加上的）
<dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-juli</artifactId>
    <version>${tomcat.version}</version>
</dependency>

这样配置打成包岂不是换个Tomcat版本就要重新打次包？ 既然是由于SpringBoot内部的Servlet容器造成了这个限制， 那我不用
<!-- 打war包时加入此项， 告诉spring-boot tomcat相关jar包用外部的，不要打进去 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

maven-war-plugin (可选) 
与maven-resources-plugin类似，当你有一些自定义的打包操作， 比如有非标准目录文件要打到war包中或者有配置文件引用了pom中的变量。 具体用法参见官方文档：http://maven.apache.org/components/plugins/maven-war-plugin/

需要继承SpringBootServletInitializer，并重写configure()方法，将Spring Boot的入口类设置进去。
// 若要部署到外部servlet容器,需要继承SpringBootServletInitializer并重写configure()
@SpringBootApplication
public class AhutApplication extends SpringBootServletInitializer{

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        // 设置启动类,用于独立tomcat运行的入口
        return builder.sources(MyWebApplication.class);
    }

}


```

- [spring data](http://projects.spring.io/spring-data/)
- [spring boot](http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)
- [spring](http://www.yiibai.com/spring/spring-tutorial-for-beginners.html)
- [spring mvc](http://www.yiibai.com/spring_mvc/springmvc_overview.html)
- [spring boot gitbook](https://www.gitbook.com/book/qbgbook/spring-boot-reference-guide-zh)
- [springfox](https://springfox.github.io/springfox/docs/current/)

