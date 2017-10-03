# 官方c++
```sh
git clone https://github.com/grpc/grpc.git
cd examples/cpp/route_guide
这个文件夹下的文件就是要用的文件，还有个route_guide.proto，注意修改Makefile
执行make之后就会生成route_guide_server与route_guide_client
```
# 官方c#
```sh
git clone https://github.com/grpc/grpc.git
cd examples/csharp/route_guide
这个文件夹下的文件就是要用的文件，还有个route_guide.proto，打开RouteGuide.sln之后还原Nuget，会下载相应的工具，修改generate_protos.bat之后就可以生成RouteGuide.cs与RouteGuideGrpc.cs两个文件，然后自己添加server与client两个项目
```
# 官方java
```sh
git clone https://github.com/grpc/grpc-java.git
cd grpc-java/examples
这个文件夹下的文件就是要用的文件，还有个route_guide.proto，要注意pom设置
进入pom所在目录执行 mvn compile
target-generated-sources-protobuf下可看到生成的文件
然后自己实现server与client
```
# 说明
```sh
官方教程可实现c++，c#，java三者互通，前提是proto要用同一个文件。
```




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

resources下建立个proto文件夹创建test.proto
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


#Ubuntu c++
```c++
apt-cache search liblssl
sudo apt-get install -y libssl-dev

examples.proto
syntax = "proto3";  
  
message SearchRequest  
{  
    string Request = 1;  
}  
  
message SearchResponse  
{  
    string Response = 2;  
}  
  
service SearchService {  
        rpc Search (SearchRequest) returns (SearchResponse);  
}  

protoc --cpp_out=./ examples.proto  
protoc --grpc_out=./ --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` examples.proto  






examples_server.cc
#include <iostream>  
#include <memory>  
#include <string>  
  
#include <grpc++/grpc++.h>  
#include <grpc/grpc.h>  
#include <grpc++/server.h>  
#include <grpc++/server_builder.h>  
#include <grpc++/server_context.h>  
  
#include "examples.grpc.pb.h"  
  
using grpc::Server;  
using grpc::ServerBuilder;  
using grpc::ServerContext;  
using grpc::Status;  
  
class SearchRequestImpl final : public SearchService::Service {  
  Status Search(ServerContext* context, const SearchRequest* request,  
                  SearchResponse* reply) override {  
    std::string prefix("Hello ");  
    reply->set_response(prefix + request->request());  
    std::cout << "Hi" << std::endl;  
    return Status::OK;  
  }  
};  
  
void RunServer() {  
  std::string server_address("0.0.0.0:50051");  
  SearchRequestImpl service;  
  
  ServerBuilder builder;  
  builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());  
  builder.RegisterService(&service);  
  std::unique_ptr<Server> server(builder.BuildAndStart());  
  std::cout << "Server listening on " << server_address << std::endl;  
  
  server->Wait();  
}  
  
int main(int argc, char** argv) {  
  RunServer();  
  
  return 0;  
}  





examples_client.cc
#include <iostream>  
#include <memory>  
#include <string>  
  
#include <grpc++/grpc++.h>  
#include <grpc/support/log.h>  
  
#include "examples.grpc.pb.h"  
  
using grpc::Channel;  
using grpc::ClientAsyncResponseReader;  
using grpc::ClientContext;  
using grpc::CompletionQueue;  
using grpc::Status;  
  
  
class ExampleClient {  
 public:  
  explicit ExampleClient(std::shared_ptr<Channel> channel)  
      : stub_(SearchService::NewStub(channel)) {}  
   
  std::string Search(const std::string& user) {  
   
    SearchRequest request;  
    request.set_request(user);  
   
    SearchResponse reply;  
     
    ClientContext context;  
  
    CompletionQueue cq;  
  
    Status status;  
  
    std::unique_ptr<ClientAsyncResponseReader<SearchResponse> > rpc(  
        stub_->AsyncSearch(&context, request, &cq));  
  
    rpc->Finish(&reply, &status, (void*)1);  
    void* got_tag;  
    bool ok = false;  
    
    GPR_ASSERT(cq.Next(&got_tag, &ok));  
  
     
    GPR_ASSERT(got_tag == (void*)1);  
    
    GPR_ASSERT(ok);  
  
    if (status.ok()) {  
      return reply.response();  
    } else {  
      return "RPC failed";  
    }  
  }  
  
 private:  
   
  std::unique_ptr<SearchService::Stub> stub_;  
};  
  
int main(int argc, char** argv) {  
  ExampleClient client(grpc::CreateChannel(  
      "localhost:50051", grpc::InsecureChannelCredentials()));  
  std::string user("world");  
  std::string reply = client.Search(user);  // The actual RPC call!  
  std::cout << "client received: " << reply << std::endl;  
  
  return 0;  
}






Makefile
subdir = ./  
  
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig  
  
SOURCES = $(wildcard $(subdir)*.cc)  
SRCOBJS = $(patsubst %.cc,%.o,$(SOURCES))  
CC = g++  
  
%.o:%.cc  
    $(CC) -std=c++11 -I/usr/local/include -pthread -c $< -o $@  
  
all: client server  
  
client: examples.grpc.pb.o examples.pb.o examples_client.o  
    $(CC) $^ -L/usr/local/lib `pkg-config --libs grpc++ grpc` -Wl,--no-as-needed -lgrpc++_reflection -Wl,--as-needed -lprotobuf -lpthread -ldl -lssl -o $@  
  
server: examples.grpc.pb.o examples.pb.o examples_server.o  
    $(CC) $^ -L/usr/local/lib `pkg-config --libs grpc++ grpc` -Wl,--no-as-needed -lgrpc++_reflection -Wl,--as-needed -lprotobuf -lpthread -ldl -lssl -o $@  
#chmod 777 $@  
  
clean:  
    sudo rm *.o  

特别说明：Makefile中$(CC)前面要用TAB，如果提示分隔符错误要重新用TAB设置锁进





运行make之后会生成server与client

```