# ubuntu
### 安装gcc
```sh
sudo apt-get install -y build-essential --fix-missing
```
### 安装tbb
```sh
sudo apt-get install -y libtbb-dev
```
### 调用动态链接库
```c++
==========test.h===========
#ifdef __cplusplus //
extern "C"
{
#endif
  
class Test{
public:
int hello(int i);
};
int helloT(int j);
#ifdef __cplusplus
}
#endif 
 
==========test.cpp===========
#include"test.h"
#include<iostream>
#include<tbb/tbb.h>
#include<tbb/parallel_for.h>
using namespace std;
using namespace tbb;  
int Test::hello(int i){
       if(i>3)
                 cout<<"hello Class Test>3"<<endl;
       else
                 parallel_for(0,10,[](int v){cout<< v <<"";});
        return 0;
}
int helloT(int j){
      Test *t=new Test();
       t->hello(j);
       return 0;
}
 
编译test.cpp文件
g++ -shared -fpic -lm -ldl -o libtest.so test.cpp -I /opt/intel/tbb/include -ltbb -std=c++11 
其中，so文件名必须以lib开头。编译具体指令请参考帮助文档
 
==========main.cpp===========
#include <stdio.h>   
#include <dlfcn.h>
#include <stdlib.h>   
#include <iostream>  
#include<tbb/tbb.h>
#include<tbb/parallel_for.h> 
using namespace std;  
using namespace tbb;  
/*
需要用到的函数
dlopen()
dlerror()
dlsym()
dlclose()
都存储在头文件dlfcn.h中
*/
int main()  {  
      
    void *handle = dlopen("./libtest.so", RTLD_LAZY);  //该处的./libtest.so表示so文件的存放位置，RTLD_LAZY是指示位
    if(!handle)  {  
        printf("open lib error\n");  
        cout<<dlerror()<<endl;  
        return -1;  
    }  
      
     typedef int (*hello)(int);//该处的函数与文件test.h中需调用的函数保持一致
     hello h= (hello)dlsym(handle, "helloT");// helloT为test.h中调用函数的名字，dlsym返回一个函数指针
     if(!h)  {  
        cout<<dlerror()<<endl; 
        dlclose(handle);  
        return -1;  
    }  
      
     int i;
     cin>>i;
    (*h)(i);//用函数指针形式调用函数
    dlclose(handle);  
    return 0;  
}  
编译main.cpp文件
g++ main.cpp -ldl -o main -I /opt/intel/tbb/include -ltbb -std=c++11 
执行./main
```