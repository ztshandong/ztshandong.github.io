# CentOS安装
```sh
yum -y install rabbitmq-server
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
http://serverip:15672/
默认用户名密码都是guest
```
# 一、pom.xml
```java
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
```
# 二、application.properties
```sh
spring.rabbitmq.host=serverip
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
# 三、创建各种类
```java
消息生产者，通过注入AmqpTemplate接口的实例来实现消息的发送，AmqpTemplate接口定义了一套针对AMQP协议的基础操作。在Spring Boot中会根据配置来注入其具体实现。在该生产者，我们会产生一个字符串，并发送到名为hello的队列中。

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.Date;

@Component
public class Sender {
    @Autowired
    private AmqpTemplate rabbitTemplate;
    public void send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        this.rabbitTemplate.convertAndSend("hello", context);
    }
}


消息接收者，通过@RabbitListener注解定义该类对hello队列的监听，并用@RabbitHandler注解来指定对消息的处理方法。所以，该消费者实现了对hello队列的消费，消费操作为输出消息的字符串内容。

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "hello")
public class Receiver {
    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver : " + hello);
    }
}



创建RabbitMQ的配置类RabbitConfig，用来配置队列、交换器、路由等高级信息。这里我们以入门为主，先以最小化的配置来定义，以完成一个基本的生产和消费过程
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class RabbitConfig {
    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }
}


单元测试
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class ServicecenterApplicationTests {
	@Autowired
	private Sender sender;
	@Test
	public void hello() throws Exception {
		sender.send();
	}
	@Test
	public void contextLoads() {
	}
}
```
# 四、测试查看结果
```java
启动应用主类，从控制台中显示如下内容，程序创建了一个访问127.0.0.1:5672中springcloud的连接。
Created new connection: SimpleConnection@3a835f3 [delegate=amqp://guest@127.0.0.1:5672/, localPort= 47292]
同时，通过RabbitMQ的控制面板，可以看到Connection和Channels中包含当前连接的条目。
运行单元测试类，我们可以看到控制台中输出下面的内容，消息被发送到了RabbitMQ Server的hello队列中。
Sender : hello Tue Apr 25 10:17:32 EDT 2017
切换到应用主类的控制台，我们可以看到类似如下输出，消费者对hello队列的监听程序执行了，并输出了接受到的消息信息。
Receiver : hello Tue Apr 25 10:17:32 EDT 2017
```