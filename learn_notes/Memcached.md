# 下载
```sh
http://libevent.org/
Debian/Ubuntu: apt-get install libevent-dev Redhat/Centos: yum install libevent-devel
./configure --with-libevent=/usr && make && sudo make install

http://memcached.org/downloads
http://memcached.org/files/memcached-1.5.6.tar.gz
./configure && make && sudo make install

http://www.runoob.com/memcached/window-install-memcached.html

memcached启动参数描述：   -d ：启动一个庇护过程，   -m：分派给Memcache利用的内存数目，单位是MB，默许是64MB，   -u ：运止Memcache的用户   -l  ：监听的办事器IP地址   -p ：设置Memcache监听的端心，默许是11211    注：-p（p为小写）   -c ：设置最年夜并发毗连数，默许是1024   -P ：设置留存Memcache的pid文件   注：-P（P为年夜写）   假设要完毕Memcache过程，执止：kill cat pid文件途径

https://github.com/junstor/memadmin

https://my.oschina.net/willSoft/blog/39646
```

# Windows安装
```sh
>1.5
schtasks /create /sc onstart /tn memcached /tr "'D:\memcached-x86\memcached.exe' -m 512"

schtasks /delete /tn memcached
```

# magent
```sh
推荐安装magent-0.5稳定版本，因为测试发现magent-0.6虽是最新版本，但是还存在问题，不稳定，第二次访问magent始终会堵塞在那里，只能set一个值。测试了magent-0.5是稳定版本，没有出现只能set一个值的现象，遂推荐安装magent-0.5版本，不过此问题也可以通过修改源码的方式来解决，解决方案参考（重点参考第8楼）：https://code.google.com/p/memagent/issues/detail?id=4#makechanges ）

关于 0.6 版本，set key 值第二次失败问题； 这个问题的关键点在于作者写 pool_server_handler 用意； 搭建环境多次启停主数据服务器(memcached)和备份服务器(memcached)；该部分源码均未被调用；该事件也未被触发； 同时因为s->sfd对应的事件机制上add了pool_server_handler的read触发条件； 影响了s->sfd接收下次set key 命令；

提供一种解决方法： 注释掉 函数 static void pool_server_handler(const int fd,const short which, void* argv)； 注释掉 put_server_into_pool(void s) 中: event_del(&(s->ev)); /注意保留/ / event_set(&(s->ev), s->sfd, EV_READ|EV_PERSIST, pool_server_handler, (void *) s); event_add(&(s->ev), 0); */ 如上方法已经测试过；原因不解释；

https://www.oschina.net/p/memagent
https://code.google.com/archive/p/memagent/downloads

https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memagent/magent-0.6.tar.gz

https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/memagent/magent-0.5.tar.gz

'SSIZE_MAX' undeclared
 vi ketama.h 
#在开头加入
#ifndef SSIZE_MAX
#define SSIZE_MAX      32767
#endif

tar -zxvf magent-0.6.tar.gz
修改Makefile文件LIBS位置
Ubuntu LIBS = -levent -lm -L/usr/local/lib

tar -zxvf magent-0.5.tar.gz
修改Makefile文件
LIBS = -levent 为LIBS = -levent -lm

############magent-0.6版本修改内容start############
vi Makefile （magent-0.6版本）

LIBS = /usr/lib64/libevent.a /usr/lib64/libm.a
改为
LIBS = -lrt /usr/lib64/libevent.a /usr/lib64/libm.a

CFLAGS = -Wall -g -O2 -I/usr/local/include $(M64)
改为
CFLAGS = -lrt -Wall -g -O2 -I/usr/local/include $(M64)
############magent-0.6版本修改内容end##############

############magent-0.5版本修改内容start############
vi Makefile （magent-0.5版本）

CFLAGS = -Wall -O2 -g
改为
CFLAGS = -lrt -Wall -O2 -g

sed -i "s#LIBS = -levent#LIBS = -levent -lm#g" Makefile
############magent-0.5版本修改内容end##############


sudo find / -name libevent-2.1.so.6
ln -s /usr/local/lib/libevent-2.1.so.6 /usr/lib/libevent-2.1.so.6

make

magent -p 11200 -s 192.168.1.164:11211 -s 192.168.1.252:11211
ps -ef | grep magent

-h this message -u uid -g gid -p port, default is 11211. (0 to disable tcp support) -s ip:port, set memcached server ip and port -b ip:port, set backup memcached server ip and port -l ip, local bind ip address, default is 0.0.0.0 -n number, set max connections, default is 4096 -D do not go to background -k use ketama key allocation algorithm -f file, unix socket path to listen on. default is off -i number, max keep alive connections for one memcached server, default is 20 -v verbose

telnet 192.168.1.164:11211
set key1 0 0 1
get key1
delete key1
```

# magent报错
```sh
http://blog.csdn.net/u011285162/article/details/28264291

报错1：

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o magent.o magent.c

magent.c: In function 'writev_list':

magent.c:729: error: 'SSIZE_MAX' undeclared (first use in this function)

magent.c:729: error: (Each undeclared identifier is reported only once

magent.c:729: error: for each function it appears in.)

make: *** [magent.o] Error 1

 

解决办法：

[root@centos6 memcached]# vi ketama.h 

#在开头加入

#ifndef SSIZE_MAX

#define SSIZE_MAX      32767

#endif

 

 

 

继续make

 

报错2：

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o magent.o magent.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o ketama.o ketama.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -o magent magent.o ketama.o /usr/lib64/libevent.a /usr/lib64/libm.a 

gcc: /usr/lib64/libevent.a: No such file or directory

gcc: /usr/lib64/libm.a: No such file or directory

 

 

解决办法：

[root@centos6 memcached]# ln -s /usr/lib/libevent*  /usr/lib64/

[root@centos6 memcached]# make

 

报错3：

gcc -Wall -g -O2 -I/usr/local/include -m64 -o magent magent.o ketama.o /usr/lib64/libevent.a /usr/lib64/libm.a 

gcc: /usr/lib64/libm.a: No such file or directory

make: *** [magent] Error 1

 

 

解决办法：

yum install glibc glibc-devel

如果是64bit的系统则不会在/usr/lib64/libm.a 生成，如果是32bit即会有。

 

[root@centos6 memcached]# cp /usr/lib64/libm.so /usr/lib64/libm.a

 

 

继续make

 

 

报错4：

gcc -Wall -g -O2 -I/usr/local/include -m64 -o magent magent.o ketama.o /usr/lib64/libevent.a /usr/lib64/libm.a 

/usr/lib64/libevent.a(event.o): In function `detect_monotonic':

event.c:(.text+0xc79): undefined reference to `clock_gettime'

/usr/lib64/libevent.a(event.o): In function `gettime':

event.c:(.text+0xd60): undefined reference to `clock_gettime'

collect2: ld returned 1 exit status

make: *** [magent] Error 1

 

 

解决办法：

[root@centos6 memcached]# vi Makefile 

CFLAGS = -Wall -g -O2 -I/usr/local/include $(M64)

改为：    

CFLAGS = -lrt -Wall -g -O2 -I/usr/local/include $(M64)
```