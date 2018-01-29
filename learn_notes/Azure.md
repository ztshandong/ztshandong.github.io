# 服务总线
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
```