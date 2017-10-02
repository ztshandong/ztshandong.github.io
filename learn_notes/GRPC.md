#java
```java
[原文](http://www.cnblogs.com/boshen-hzb/p/6555221.html)

新建maven项目
<groupId>com.mingluck.test</groupId>
<artifactId>grpc</artifactId>

pom
<dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-all</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.4.2</version>
        </dependency>
    </dependencies>

    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.4.1.Final</version>
            </extension>
        </extensions>
        <plugins>
            <!-- protobuf -->
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.0</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.0.0-beta-2:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:0.14.0:exe:${os.detected.classifier}</pluginArtifact>
                    <protoSourceRoot>src/main/resources/proto</protoSourceRoot>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

test.proto
syntax = "proto3";
package grpc;
option java_package = "com.mingluck.grpc";
option java_outer_classname = "HelloWorldServiceProto";
option java_multiple_files = true;

//服务端接口类
service Greeter {
  //服务端接口方法
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

//请求参数
message HelloRequest {
  string name = 1;
  string sex = 2;
}

//响应参数
message HelloReply {
  string message = 1;
}

进入pom所在目录执行 mvn compile
target-generated-sources-protobuf下可看到生成的文件



HelloWorldServer.java
package com.mingluck.grpc;
/**
 * Created by Darren on 2016/11/11.
 */
import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;

import java.io.IOException;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.logging.Logger;

public class HelloWorldServer {
    private static final Logger logger = Logger.getLogger(HelloWorldServer.class.getName());

    /* The port on which the server should run */
    private int port = 50051;
    private Server server;

    private void start() throws IOException {
        server = ServerBuilder.forPort(port)
                .addService(GreeterGrpc.bindService(new GreeterImpl()))
                .build()
                .start();
        logger.info("Server started, listening on " + port);
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                // Use stderr here since the logger may have been reset by its JVM shutdown hook.
                System.err.println("*** shutting down gRPC server since JVM is shutting down");
                HelloWorldServer.this.stop();
                System.err.println("*** server shut down");
            }
        });
    }

    private void stop() {
        if (server != null) {
            server.shutdown();
        }
    }

    /**
     * Await termination on the main thread since the grpc library uses daemon threads.
     */
    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }

    /**
     * Main launches the server from the command line.
     */
    public static void main(String[] args) throws IOException, InterruptedException {
        final HelloWorldServer server = new HelloWorldServer();
        server.start();
        server.blockUntilShutdown();
    }

    private class GreeterImpl implements GreeterGrpc.Greeter {
        /** 原子Integer */
        public AtomicInteger count = new AtomicInteger(0);

        @Override
        public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
            System.out.println("call sayHello");
            HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName() + req.getSex()).build();
            responseObserver.onNext(reply);
            responseObserver.onCompleted();
            System.out.println(count.incrementAndGet() + Thread.currentThread().getName());
        }
    }
}



HelloWorldClient.java
package com.mingluck.grpc;

/**
 * Created by Darren on 2016/11/11.
 */
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;
import java.util.concurrent.TimeUnit;
import java.util.logging.Level;
import java.util.logging.Logger;
/**
 * A simple client that requests a greeting from the {@link HelloWorldServer}.
 */
public class HelloWorldClient {
    private static final Logger logger = Logger.getLogger(HelloWorldClient.class.getName());

    private final ManagedChannel channel;
    private final GreeterGrpc.GreeterBlockingStub blockingStub;

    /** Construct client connecting to HelloWorld server at {@code host:port}. */
    public HelloWorldClient(String host, int port) {

        channel = ManagedChannelBuilder.forAddress(host, port)
                .usePlaintext(true)
                .build();
        blockingStub = GreeterGrpc.newBlockingStub(channel);
    }

    public void shutdown() throws InterruptedException {
        channel.shutdown().awaitTermination(5, TimeUnit.SECONDS);
    }

    /** Say hello to server. */
    public void greet(String name) {
        logger.info("Will try to greet " + name + " ...");
        HelloRequest request = HelloRequest.newBuilder().setName(name).setSex(" 女").build();
        HelloReply response;
        try {
            response = blockingStub.sayHello(request);
        } catch (StatusRuntimeException e) {
            logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
            return;
        }
        logger.info("Greeting: " + response.getMessage());
    }

    /**
     * Greet server. If provided, the first element of {@code args} is the name to use in the
     * greeting.
     */
    public static void main(String[] args) throws Exception {
        HelloWorldClient client = new HelloWorldClient("localhost", 50051);
        try {

            String user = "world";
            if (args.length > 0) {
                user = args[0];
            }
            client.greet(user);
        } finally {
           client.shutdown();
        }
    }
}





为了防止客户端不断调用带来的开销(短连接)，下面的例子给出了连接池的方式
package com.mingluck.grpc;
import org.apache.commons.pool2.BasePooledObjectFactory;
import org.apache.commons.pool2.PooledObject;
import org.apache.commons.pool2.impl.DefaultPooledObject;
import org.apache.commons.pool2.impl.GenericObjectPool;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;

/**
 * Created by darren on 2016/11/14.
 */
public class HelloWorldClientFactory extends BasePooledObjectFactory<HelloWorldClient> {

    @Override
    public HelloWorldClient create() throws Exception {
        return new HelloWorldClient("localhost", 50051);
    }

    @Override
    public PooledObject<HelloWorldClient> wrap(HelloWorldClient client) {
        return new DefaultPooledObject<HelloWorldClient>(client);
    }

    @Override
    public void destroyObject(PooledObject<HelloWorldClient> p) throws Exception {
        HelloWorldClient client = p.getObject();
        client.shutdown();
        super.destroyObject(p);
    }

    public static void main(String[] args) throws Exception {

        /** 连接池的配置 */
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();

        /** 下面的配置均为默认配置,默认配置的参数可以在BaseObjectPoolConfig中找到 */
        poolConfig.setMaxTotal(8); // 池中的最大连接数
        poolConfig.setMinIdle(0); // 最少的空闲连接数
        poolConfig.setMaxIdle(8); // 最多的空闲连接数
        poolConfig.setMaxWaitMillis(-1); // 当连接池资源耗尽时,调用者最大阻塞的时间,超时时抛出异常 单位:毫秒数
        poolConfig.setLifo(true); // 连接池存放池化对象方式,true放在空闲队列最前面,false放在空闲队列最后
        poolConfig.setMinEvictableIdleTimeMillis(1000L * 60L * 30L); // 连接空闲的最小时间,达到此值后空闲连接可能会被移除,默认即为30分钟
        poolConfig.setBlockWhenExhausted(true); // 连接耗尽时是否阻塞,默认为true

        /** 连接池创建 */
        GenericObjectPool<HelloWorldClient> objectPool = new GenericObjectPool<HelloWorldClient>(new HelloWorldClientFactory(), poolConfig);


        new Thread(makeTask(objectPool)).start();
        new Thread(makeTask(objectPool)).start();
        new Thread(makeTask(objectPool)).start();
        new Thread(makeTask(objectPool)).start();

        Thread.sleep(100000);

    }

    private static Runnable makeTask(GenericObjectPool<HelloWorldClient> objectPool){
        return () -> {
            HelloWorldClient client = null;
            try {
                client = objectPool.borrowObject();

            } catch (Exception e) {
                e.printStackTrace();
            }
            try {
                String req = "world!";
                client.greet(req);
            } finally {
                /** 将连接对象返回给连接池 */
               objectPool.returnObject(client);
            }
        };
    }
}
```