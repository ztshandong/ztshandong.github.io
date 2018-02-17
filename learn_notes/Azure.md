# vs登录Azure
```sh
https://docs.azure.cn/zh-cn/articles/azure-operations-guide/others/aog-portal-management-qa-vs2015-login

https://docs.azure.cn/zh-cn/articles/azure-operations-guide/others/aog-portal-management-qa-vs2017-login
安装扩展Azure Environment Selector
Tools-Azure Environment Selector
```

# IDEA登录Azure
```sh
安装Azure Toolkit for IntelliJ
在~/AzureToolsForIntelliJ目录下创建文件AadProvider.json
{
"EnvironmentName": "CHINA"
}
```

# 服务总线队列
```java
服务总线，添加队列
		<dependency>

			<groupId>com.microsoft.azure</groupId>

			<artifactId>azure-servicebus</artifactId>

			<version>0.9.7</version>

		</dependency>

static void createQueue(String queueName){

		Configuration config =

				ServiceBusConfiguration.configureWithSASAuthentication(

						"服务总线名称",

						"RootManageSharedAccessKey",

						"主密钥",

						".servicebus.chinacloudapi.cn"

				);

		ServiceBusContract service = ServiceBusService.create(config);

		QueueInfo queueInfo = new QueueInfo(queueName);

		try

		{

			CreateQueueResult result = service.createQueue(queueInfo);

		}

		catch (ServiceException e)

		{

			System.out.print("ServiceException encountered: ");

			System.out.println(e.getMessage());

			System.exit(-1);

		}

	}


.NetCore项目安装Microsoft.Azure.ServiceBus
发送方
using System;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Azure.ServiceBus;

namespace AzureServiceBusSender
{
    class Program
    {
        const string ServiceBusConnectionString = "主连接字符串";
        const string QueueName = "队列名称";
        static IQueueClient queueClient;
        static void Main(string[] args)
        {
            MainAsync().GetAwaiter().GetResult();
        }
        static async Task MainAsync()
        {
            const int numberOfMessages = 10;
            queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

            Console.WriteLine("======================================================");
            Console.WriteLine("Press ENTER key to exit after sending all the messages.");
            Console.WriteLine("======================================================");

            // Send messages.
            await SendMessagesAsync(numberOfMessages);

            Console.ReadKey();

            await queueClient.CloseAsync();
        }
        static async Task SendMessagesAsync(int numberOfMessagesToSend)
        {
            try
            {
                for (var i = 0; i < numberOfMessagesToSend; i++)
                {
                    // Create a new message to send to the queue.
                    string messageBody = $"Message {i}";
                    var message = new Message(Encoding.UTF8.GetBytes(messageBody));

                    // Write the body of the message to the console.
                    Console.WriteLine($"Sending message: {messageBody}");

                    // Send the message to the queue.
                    await queueClient.SendAsync(message);
                }
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
            }
        }
    }
}




接收方
namespace AzureServiceBusReceiver
{
    using System;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;

    class Program
    {
        // Connection String for the namespace can be obtained from the Azure portal under the 
        // 'Shared Access policies' section.
        const string ServiceBusConnectionString = "主连接字符串";
        const string QueueName = "队列名称";
        static IQueueClient queueClient;

        static void Main(string[] args)
        {
            MainAsync().GetAwaiter().GetResult();
        }

        static async Task MainAsync()
        {
            queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

            Console.WriteLine("======================================================");
            Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
            Console.WriteLine("======================================================");

            // Register QueueClient's MessageHandler and receive messages in a loop
            RegisterOnMessageHandlerAndReceiveMessages();

            Console.ReadKey();

            await queueClient.CloseAsync();
        }

        static void RegisterOnMessageHandlerAndReceiveMessages()
        {
            // Configure the MessageHandler Options in terms of exception handling, number of concurrent messages to deliver etc.
            var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
            {
                // Maximum number of Concurrent calls to the callback `ProcessMessagesAsync`, set to 1 for simplicity.
                // Set it according to how many messages the application wants to process in parallel.
                MaxConcurrentCalls = 1,

                // Indicates whether MessagePump should automatically complete the messages after returning from User Callback.
                // False below indicates the Complete will be handled by the User Callback as in `ProcessMessagesAsync` below.
                AutoComplete = false
            };

            // Register the function that will process messages
            queueClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
        }

        static async Task ProcessMessagesAsync(Message message, CancellationToken token)
        {
            // Process the message
            Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

            // Complete the message so that it is not received again.
            // This can be done only if the queueClient is created in ReceiveMode.PeekLock mode (which is default).
            await queueClient.CompleteAsync(message.SystemProperties.LockToken);

            // Note: Use the cancellationToken passed as necessary to determine if the queueClient has already been closed.
            // If queueClient has already been Closed, you may chose to not call CompleteAsync() or AbandonAsync() etc. calls 
            // to avoid unnecessary exceptions.
        }

        static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
        {
            Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
            var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
            Console.WriteLine("Exception context for troubleshooting:");
            Console.WriteLine($"- Endpoint: {context.Endpoint}");
            Console.WriteLine($"- Entity Path: {context.EntityPath}");
            Console.WriteLine($"- Executing Action: {context.Action}");
            return Task.CompletedTask;
        }
    }
}
```

# 服务总线-主题订阅
```c#
Azure门户新增主题时如果选择启用重复检测，貌似订阅接收时有序，否则接收无序。
新增订阅时不要勾选启用会话，否则订阅接收报错。

.NetCore项目安装Microsoft.Azure.ServiceBus

主题
namespace AzureServiceBusZhuTi
{
    using System;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;

    class Program
    {
        const string ServiceBusConnectionString = "主连接字符串";
        const string TopicName = "主题名";
        static ITopicClient topicClient;

        static void Main(string[] args)
        {
            MainAsync().GetAwaiter().GetResult();
        }

        static async Task MainAsync()
        {
            const int numberOfMessages = 25;
            topicClient = new TopicClient(ServiceBusConnectionString, TopicName);

            Console.WriteLine("======================================================");
            Console.WriteLine("Press ENTER key to exit after sending all the messages.");
            Console.WriteLine("======================================================");

            // Send messages.
            await SendMessagesAsync(numberOfMessages);

            Console.ReadKey();

            await topicClient.CloseAsync();
        }

        static async Task SendMessagesAsync(int numberOfMessagesToSend)
        {
            try
            {
                for (var i = 0; i < numberOfMessagesToSend; i++)
                {
                    // Create a new message to send to the topic
                    string messageBody = $"Message {i}";
                    var message = new Message(Encoding.UTF8.GetBytes(messageBody));
                    //创建主题的时候选择了启用重复检查需要设置PartitionKey
                    //message.PartitionKey = "1";
                    // Write the body of the message to the console
                    Console.WriteLine($"Sending message: {messageBody}");

                    // Send the message to the topic
                    await topicClient.SendAsync(message);
                }
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
            }
        }
    }
}


namespace CoreReceiverApp
{
    using System;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;

    class Program
    {
        const string ServiceBusConnectionString = "主连接字符串";
        const string TopicName = "主题名称";
        const string SubscriptionName = "订阅名称";
        static ISubscriptionClient subscriptionClient;

        static void Main(string[] args)
        {
            MainAsync().GetAwaiter().GetResult();
        }

        static async Task MainAsync()
        {
            subscriptionClient = new SubscriptionClient(ServiceBusConnectionString, TopicName, SubscriptionName);

            Console.WriteLine("======================================================");
            Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
            Console.WriteLine("======================================================");

            // Register subscription message handler and receive messages in a loop.
            RegisterOnMessageHandlerAndReceiveMessages();

            Console.ReadKey();

            await subscriptionClient.CloseAsync();
        }

        static void RegisterOnMessageHandlerAndReceiveMessages()
        {
            // Configure the message hnadler options in terms of exception handling, number of concurrent messages to deliver, etc.
            var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
            {
                // Maximum number of concurrent calls to the callback ProcessMessagesAsync(), set to 1 for simplicity.
                // Set it according to how many messages the application wants to process in parallel.
                MaxConcurrentCalls = 1,

                // Indicates whether MessagePump should automatically complete the messages after returning from User Callback.
                // False below indicates the Complete will be handled by the User Callback as in `ProcessMessagesAsync` below.
                AutoComplete = false
            };

            // Register the function that processes messages.
            subscriptionClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
        }

        static async Task ProcessMessagesAsync(Message message, CancellationToken token)
        {
            // Process the message.
            Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

            // Complete the message so that it is not received again.
            // This can be done only if the subscriptionClient is created in ReceiveMode.PeekLock mode (which is the default).
            await subscriptionClient.CompleteAsync(message.SystemProperties.LockToken);

            // Note: Use the cancellationToken passed as necessary to determine if the subscriptionClient has already been closed.
            // If subscriptionClient has already been closed, you can choose to not call CompleteAsync() or AbandonAsync() etc.
            // to avoid unnecessary exceptions.
        }

        static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
        {
            Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
            var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
            Console.WriteLine("Exception context for troubleshooting:");
            Console.WriteLine($"- Endpoint: {context.Endpoint}");
            Console.WriteLine($"- Entity Path: {context.EntityPath}");
            Console.WriteLine($"- Executing Action: {context.Action}");
            return Task.CompletedTask;
        }
        
    }
}

```

# Spark
```sh
Azure门户
spark群集-存储账户-容器-上传
上传helloSpark.txt
hello spark
hello world
中文

ssh登录后
spark-shell

val lines=sc.textFile("/helloSpark.txt")
val linesflatMap=lines.flatMap(line=>line.split(" "))
lines.count()
lines.first()
lines.filter(line => line.contains("hello")).collect().foreach(println)

如果执行
val lines=sc.textFile("helloSpark.txt").count()
会到路径/user/sshuser/下

val rdd=sc.parallelize(Array(1,2,2,4),4)
rdd.collect().foreach(println)
rdd.take(4).foreach(println)
rdd.toDF().show()

val rdd = sc.parallelize(1 to 10)
rdd.reduce((x,y)=>x+y)
val map = rdd.map(_*2)
rdd.top(2)

val rdd2=sc.parallelize(Array((1,2),(1,3),(2,5),(2,6)))
rdd2.reduceByKey((x,y)=>x+y).collect().foreach(println)
rdd2.groupByKey().collect().foreach(println)
rdd2.mapValues(x=>x+1).collect().foreach(println)
rdd2.flatMapValues(x=>(x to 5)).collect().foreach(println)
rdd2.keys.collect().foreach(println)
rdd2.values.collect().foreach(println)
rdd2.sortByKey().collect().foreach(println)

val lines=sc.parallelize(Array("hello","spark","hello","world","!"))
val linemap=lines.map(word=>(word,1))
lines.distinct().collect().foreach(println)

val lines2=sc.parallelize(Array("hello","hadoop","hello","world"))

lines.union(lines2).collect().foreach(println)
lines.intersection(lines2).collect().foreach(println)
lines.subtract(lines2).collect().foreach(println)  lines有，lines2没有的

```

# Spark-Maven
```java
新建Maven项目，删掉java，resource，test
main下新建scala文件夹，右键scala，Make Directory as Sources Root
项目右键Add Framework Support-Scala-2.11.12

pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>scala</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>scala</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spark.version>2.0.2</spark.version>
		<scala.version>2.11</scala.version>
	</properties>

	<dependencies>
		<!--<dependency>-->
			<!--<groupId>org.springframework.boot</groupId>-->
			<!--<artifactId>spring-boot-starter-web</artifactId>-->
		<!--</dependency>-->

		<!--<dependency>-->
			<!--<groupId>org.springframework.boot</groupId>-->
			<!--<artifactId>spring-boot-starter-test</artifactId>-->
			<!--<scope>test</scope>-->
		<!--</dependency>-->

		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-core_${scala.version}</artifactId>
			<version>${spark.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-streaming_${scala.version}</artifactId>
			<version>${spark.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-sql_${scala.version}</artifactId>
			<version>${spark.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-hive_${scala.version}</artifactId>
			<version>${spark.version}</version>
		</dependency>
		<dependency>
			<groupId>org.apache.spark</groupId>
			<artifactId>spark-mllib_${scala.version}</artifactId>
			<version>${spark.version}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<!--<plugin>-->
				<!--<groupId>org.springframework.boot</groupId>-->
				<!--<artifactId>spring-boot-maven-plugin</artifactId>-->
			<!--</plugin>-->

			<plugin>
				<groupId>org.scala-tools</groupId>
				<artifactId>maven-scala-plugin</artifactId>
				<version>2.15.2</version>
				<executions>
					<execution>
						<goals>
							<goal>compile</goal>
							<goal>testCompile</goal>
						</goals>
					</execution>
				</executions>
			</plugin>

			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.6.0</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.19</version>
				<configuration>
					<skip>true</skip>
				</configuration>
			</plugin>

		</plugins>
	</build>
</project>



scala文件夹右键，new Scala Class，kind-Object
import org.apache.spark.{SparkConf, SparkContext}
/**
  * Created by zhangtao on 2018/1/31.
  */
object myFirstScala {
//  def main(args: Array[String]):Unit = {
//    println("Hello World!")
//  }

  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("myFirstScala")
    //setMaster("local") 本机的spark就用local，远端的就写ip
    //如果是打成jar包运行则需要去掉 setMaster("local")因为在参数中会指定。
    conf.setMaster("local")
/*
local 本地单线程
local[K] 本地多线程（指定K个内核）
local[*] 本地多线程（指定所有可用内核）
spark://HOST:PORT 连接到指定的 Spark standalone cluster master，需要指定端口。
mesos://HOST:PORT 连接到指定的 Mesos 集群，需要指定端口。
yarn-client客户端模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。
yarn-cluster集群模式 连接到 YARN 集群。需要配置 HADOOP_CONF_DIR。
 */
    val sc =new SparkContext(conf)
    val rdd =sc.parallelize(List(1,2,3,4,5,6)).map(_*3)
    val mappedRDD=rdd.filter(_>10).collect()
    //对集合求和
    println(rdd.reduce(_+_))
    //输出大于10的元素
    for(arg <- mappedRDD)
      print(arg+" ")
    println()
    println("math is work")
  }
}



“File“然后选择“project Structure“
Artifacts-JAR-From modules with dependencies
删除其他无用包，只留mySpark.jar和'mySpark'compile output
Build Artifacts
```

# Media
```sh
实时传送视频流很不稳定
资产-上传-发布-编码
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.dashjs.org/latest/dash.all.min.js"></script>
    <style>
        video {
            width: 640px;
            height: 360px;
        }
    </style>
</head>
<body>
<div>
    <video data-dashjs-player autoplay src="MPEG-DASH URL" controls></video>
</div>
</body>
</html>
```
