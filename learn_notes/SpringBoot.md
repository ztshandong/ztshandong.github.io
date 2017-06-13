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

- [spring data](http://projects.spring.io/spring-data/)
- [spring boot](http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)
- [spring](http://www.yiibai.com/spring/spring-tutorial-for-beginners.html)
- [spring mvc](http://www.yiibai.com/spring_mvc/springmvc_overview.html)
- [spring boot gitbook](https://www.gitbook.com/book/qbgbook/spring-boot-reference-guide-zh)
- [springfox](https://springfox.github.io/springfox/docs/current/)

