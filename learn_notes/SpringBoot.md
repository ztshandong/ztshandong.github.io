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
- /src/main/resources/application使用yml格式，否则中文有乱码
```sh
部署方法之一，在项目路径下
mvn install
会在target目录下新建一个jar包
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

- [maven数据库](maven repository)
- [spring data](http://projects.spring.io/spring-data/)
- [spring cloud](http://bbs.springcloud.cn/)
- [spring boot](http://blog.didispace.com/Spring-Boot%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)
- [spring cloud](http://blog.didispace.com/Spring-Cloud%E5%9F%BA%E7%A1%80%E6%95%99%E7%A8%8B/)
- [spring](http://www.yiibai.com/spring/spring-tutorial-for-beginners.html)
- [spring mvc](http://www.yiibai.com/spring_mvc/springmvc_overview.html)
- [spring boot gitbook](https://www.gitbook.com/book/qbgbook/spring-boot-reference-guide-zh)
- [springfox](https://springfox.github.io/springfox/docs/current/)

