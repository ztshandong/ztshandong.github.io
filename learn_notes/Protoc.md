#idea
```java
Plugins里搜索安装protobuf support
main下面新建proto文件夹，把xxx.proto放入
pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->

	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<!--grpc版本号-->
		<grpc.version>1.7.0</grpc.version>
		<!--protobuf 版本号-->
		<protobuf.version>3.5.1</protobuf.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.google.protobuf</groupId>
			<artifactId>protobuf-java</artifactId>

		</dependency>

		<dependency>
			<groupId>com.google.protobuf</groupId>
			<artifactId>protobuf-java</artifactId>
			<version>${protobuf.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-netty</artifactId>
			<version>${grpc.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-protobuf</artifactId>
			<version>${grpc.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-stub</artifactId>
			<version>${grpc.version}</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>

	<build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>

            <plugin>

                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}
                    </protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}}:exe:${os.detected.classifier}
                    </pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
                <!--<groupId>org.springframework.boot</groupId>-->
                <!--<artifactId>spring-boot-maven-plugin</artifactId>-->
            </plugin>
        </plugins>
    </build>

</project>


mvn package 会生成xxx.java文件
```

# Ubuntu安装protobuf
```sh
https://github.com/google/protobuf/releases/

Linux
sudo apt-get install -y build-essential autoconf libtool
sudo apt-get install -y libgflags-dev libgtest-dev
sudo apt-get install -y clang libc++-dev
sudo apt-get install -y pkg-config

macOS
sudo xcode-select --install
brew install autoconf automake libtool shtool
brew install gflags

Ubuntu下只用过c++的，后面grpc要用which grpc_cpp_plugin
https://github.com/google/protobuf/releases/download/v3.4.1/protobuf-cpp-3.4.1.tar.gz
tar -zxvf v3.4.1.tar.gz
cd protobuf-3.4.1

如果下的是https://github.com/google/protobuf/archive/v3.4.1.tar.gz就要先./autogen.sh

./configure

MacOS好像要用LIBTOOL=glibtool LIBTOOLIZE=glibtoolize make
make
make check
sudo make install

protoc --version

protobuf的默认安装路径是/usr/local/lib，而/usr/local/lib 不在Ubuntu体系默认的 LD_LIBRARY_PATH 里，所以就找不到该lib 
sudo vi /etc/ld.so.conf.d/libprotobuf.conf
/usr/local/lib

sudo ldconfig

MacOS貌似brew install protobuf即可

```
#Ubuntu安装grpc
```sh
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc
git submodule update --init
make
sudo make install
```