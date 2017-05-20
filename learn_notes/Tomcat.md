
# 配置api，开发环境为eclipse mars + jdk 7 + tomcat 7
```sh 
一、将服务器上tomcat/webapps文件夹清空，再建一个app1name文件夹
二、新建eclipse项目使用默认输出路径build\classes，最后一步WebContent改名为WebRoot
三、复制自己WebRoot下所有文件，拷到服务器的app1name文件夹下，相当于将自己的WebRoot文件夹改名为app1name，然后拷到服务器的tomcat/webapps文件夹下
四、复制自己的build/classes文件夹，拷到服务器app1name/WEB-INF文件夹里面
五、以后修改java代码只要复制build/classes即可，
```
# 配置tomcat/conf/server.xml，需要配合host文件
# 如果要是java的api后台，建议使用单独的tomcat，如果是纯静态的web，可以公用一个tomcat
```sh
将默认Host内容注释掉
 <Host name="app1name" appBase="webapps" unpackWARs="false" autoDeploy="false" xmlValidation="false" xmlNamespaceAware="false">
      	<Context path="" docBase="app1name" reloadable="true" caseSensitive="false" debug="0"></Context>
     	</Host>
 <Host name="app2name" appBase="webapps" unpackWARs="false" autoDeploy="false" xmlValidation="false" xmlNamespaceAware="false">
      	<Context path="" docBase="app2name" reloadable="true" caseSensitive="false" debug="0"></Context>
     	</Host>
 <Host name="app3name" appBase="webapps" unpackWARs="false" autoDeploy="false" xmlValidation="false" xmlNamespaceAware="false">
      	<Context path="" docBase="app3name" reloadable="true" caseSensitive="false" debug="0"></Context>
     	</Host>
```

# MAC
```sh
brew install tomcat-native
==> Installing dependencies for tomcat-native: openssl, apr
==> Installing tomcat-native dependency: openssl
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
######################################################################## 100.0%
==> Pouring openssl-1.0.2k.sierra.bottle.tar.gz
==> Using the sandbox
==> Caveats
A CA file has been bootstrapped using certificates from the SystemRoots
keychain. To add additional certificates (e.g. the certificates added in
the System keychain), place .pem files in
  /usr/local/etc/openssl/certs

and run
  /usr/local/opt/openssl/bin/c_rehash

This formula is keg-only, which means it was not symlinked into /usr/local.

Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries

If you need to have this software first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile

For compilers to find this software you may need to set:
    LDFLAGS:  -L/usr/local/opt/openssl/lib
    CPPFLAGS: -I/usr/local/opt/openssl/include

==> Summary
🍺  /usr/local/Cellar/openssl/1.0.2k: 1,696 files, 12MB
==> Installing tomcat-native dependency: apr
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ap
######################################################################## 100.0%
==> Pouring apr-1.5.2_3.sierra.bottle.tar.gz
==> Caveats
This formula is keg-only, which means it was not symlinked into /usr/local.

Apple's CLT package contains apr

If you need to have this software first in your PATH run:
  echo 'export PATH="/usr/local/opt/apr/bin:$PATH"' >> ~/.bash_profile

==> Summary
🍺  /usr/local/Cellar/apr/1.5.2_3: 56 files, 1.2MB
==> Installing tomcat-native 
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/to
######################################################################## 100.0%
==> Pouring tomcat-native-1.2.10.sierra.bottle.tar.gz
==> Caveats
In order for tomcat's APR lifecycle listener to find this library, you'll
need to add it to java.library.path. This can be done by adding this line
to $CATALINA_HOME/bin/setenv.sh

  CATALINA_OPTS="$CATALINA_OPTS -Djava.library.path=/usr/local/opt/tomcat-native/lib"

If $CATALINA_HOME/bin/setenv.sh doesn't exist, create it and make it executable.
==> Summary
🍺  /usr/local/Cellar/tomcat-native/1.2.10: 11 files, 376.2KB
```
