# 编译方式
```
windows下
c语言用#define MYLIBAPIC  extern _declspec( dllexport ) 
c++用#define MYLIBAPI  extern "C" __declspec( dllexport ) 
linux下
c语言用#define MYLIBAPIC  extern  
c++用#define MYLIBAPI  extern "C" 


g++ add.h add_c.h add.cpp add_c.c -fPIC -shared -o libC2CPLUSTest.so

 g++ add.cpp add_c.c -fPIC -shared -o libC2CPLUSTest.so

linux编译命令
gcc add.cpp add_c.c add_cpp2c.cpp -lstdc++ -fPIC -shared -o libC2CPLUSTest.so 


vi /etc/profile
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$/root

source /etc/profile

```

