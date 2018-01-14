# 程序集混淆签名
```sh
所有命令都在vs2015开发人员工具（管理员权限）下运行，区分大小写。
一、创建密钥
sn -k csharp.snk
sn -p csharp.snk csharp.public.snk
二、对C++项目进行强签名
1、在cpp文件构造函数上面添加三行
using namespace System::Reflection;
[assembly:AssemblyKeyFileAttribute("cppkey.snk")];
[assembly:AssemblyDelaySignAttribute(true)];

大概样式如下
#include "test5.h"
using namespace System::Reflection;
[assembly:AssemblyKeyFileAttribute("cppkey.snk")];
[assembly:AssemblyDelaySignAttribute(true)];
test5::test5(void)
{
}

2、项目属性-生成事件-预先生成事件-命令行：sn -k $(OutDir)\cppkey.snk，则编译后会生成cppkey.snk文件

3、生成DLL

4、sn -Ra cpp.dll cppkey.snk，然后就可以给其他项目使用了

三、对C#项目进行强签名
1、c#所有项目属性中选择为程序集签名与延迟签名，选择csharp.public.snk
2、延迟签名后程序不能直接运行，需要使用-Vr免验证Hash
sn -Vr test.dll
sn -Vr test.exe

3、使用Dotfuscator混淆C#的DLL与exe
Settings-ProjectProperties需要添加两个参数
ILASM_v4.0.30319
C:\Windows\Microsoft.NET\Framework\v4.0.30319\ilasm.exe

ILDASM_v4.0.30319
C:\Program Files (x86)\Microsoft SDKs\Windows\v8.1A\bin\NETFX 4.5.1 Tools\ildasm.exe

DisableStringEncryption改为No
RenameOptions选择UseEnhancedOverloadInductiong
RenamingSchema选择Unprintable

4、对混淆后的文件进行签名
sn -Ra test.dll csharp.snk
sn -Ra test.exe csharp.snk

注意事项：
一、c++项目进行强签名需要将密钥文件名（扩展名为snk）写入到代码中，并且每一次生成项目都会使snk发生变化，所以c++项目与c#项目绝对不要使用同一个密钥，否则会签名失败。
二、强签名的项目所引用的所有DLL也必须为强签名的。
三、C++的DLL使用Dotfuscator混淆会报错。

csharp.public.snk是提取出来的公钥，给开发人员使用。
csharp.snk是公私钥对，要由专人保管。
```

# Mac DotNet
```sh
dotnet: command not found
打开新终端，如果还不行就执行下面两句
ln -s /usr/local/share/dotnet/bin/dotnet /usr/local/bin/
ln -s /usr/local/share/dotnet/bin/csc /usr/local/bin/
```

# DotNET Core Ubuntu 16.04
```sh
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg

sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list'

sudo apt-get update
sudo apt-get install dotnet-sdk-2.0.2 -y
```

# DotNet Core WPF
```sh
windows下可安装Avalonia.vsix
git clone https://github.com/AvaloniaUI/Avalonia.git
git submodule update --init

linux/osx
git clone https://github.com/AvaloniaUI/Avalonia.git
git submodule update --init --recursive
samples/ControlCatalog.NetCore
dotnet restore
dotnet run
```

# windows调用c++
```sh
Cpp项目

DLL.h
extern"C" _declspec(dllexport) char* strcpyTest(char* dest, char* sour);

extern"C" _declspec(dllexport) int Test2(char* flowno, char* salesman, int offlinenum, char* offlinegoods, int onlinenum, char* onlinegoods, char** password, char** memo);

extern"C" _declspec(dllexport) int Test3(char* c);

extern"C" _declspec(dllexport) void Test4();

extern"C" _declspec(dllexport) void Test5();

extern"C" _declspec(dllexport) int Test6();

DLL.cpp
#include "stdafx.h"  
#include "DLL.h"  
#include<stdio.h>
char* strcpyTest(char* dest, char* sour)
{
	char* temp = dest;
	while ('\0' != *sour)
	{
		*dest = *sour;
		dest++;
		sour++;
	}
	*dest = '\0';
	return temp;
}

int Test2(char* flowno, char* salesman, int offlinenum, char* offlinegoods, int onlinenum, char* onlinegoods, char** password, char** memo)
{
	printf("salesman = %s\n", salesman);
	printf("&salesman = 0x%x\n", &salesman);

	printf("password = %s\n", password);
	printf("&password = 0x%x\n", &password);
	return 9527;
}




class CPointer
{
public:
	CPointer() {};
	~CPointer() {};
	int	Cptest() { return 123; };
public:
	static char * m_p;
};

static CPointer *cp;

int Test6()
{
	for (int i = 0; i < 10; ++i)
	{
		if (NULL != cp)
		{
			printf("del %d\n", i);
			cp= NULL;
			delete[]cp;
		}
		printf("%d\n", i);
		cp = new CPointer;
		printf("%d\n", cp->Cptest()); 
	}
	return 8341;
}

char * CPointer::m_p = nullptr;

int Test3(char* c)
{
	for (int i = 0; i < 10; ++i)
	{
		if (nullptr != CPointer::m_p)
		{
			//char * p = CPointer::m_p;
			//delete[]p;
			delete[]CPointer::m_p;
			//p = nullptr;
			//CPointer::m_p = nullptr;
		}
		printf("%d\n", i);
		printf("%s\n", c);
		CPointer::m_p = new char[1024 * 1024];
	}
	return 8341;
}



void Test4() {

	int i = 5;
	int *p = &i;
	int j = (int)p;

	printf("\n");
	printf("int i = 5;\n");
	printf("int *p = &i;\n");
	printf("int j = (int)p;\n");
	printf("\n");

	printf("i= %d, &i= 0x%x\n", i, &i);
	printf("&p=: 0x%x, Value of P is: 0x%x=%d, *p=: %d\n", &p, p, p, *p);
	if (p == &i)
		printf("p == &i\n");
	printf("j = %d\n", j);
	printf("j = 0x%x\n", j);
	//printf("*(int*)j=: %d\n", *(int*)j);

	volatile uintptr_t iptr = j;
	unsigned int *ptr = (unsigned int *)iptr;

	printf("ptr= 0x%x\n", ptr);
}


void Test5() {
	int n = 5;
	char c = 'A';
	void *p = &n;
	p = &c;
	short *p2 = (short*)p;
	printf("\n");
	printf("p= 0x%x\n", p);
	printf("p2= 0x%x\n", p2);

	
}


unsigned int *g(void) {

	int i = 5;
	int *p = &i;
	int j = (int)p;

	volatile uintptr_t iptr = j;
	unsigned int *ptr = (unsigned int *)iptr;
	
	printf("Value of ptr is 0x%x\n", ptr);
	printf("Value of ptr is %d\n", *ptr);


	return ptr;
}

.NetCore项目
using System;
using System.Runtime.InteropServices;

namespace NetCore
{
    class Program
    {
        [DllImport("Cpp.dll", EntryPoint = "strcpyTest", CallingConvention = CallingConvention.Cdecl)]
        public static extern IntPtr strcpyTest(ref byte dest, string sour);


        [DllImport("Cpp.dll", EntryPoint = "Test2" )]//CharSet = CharSet.Ansi
        public static extern IntPtr Test2(string flowno, string salesman, int offlinenum, string offlinegoods, int onlinenum, string onlinegoods, ref string password, ref string memo);

        [DllImport("Cpp.dll", EntryPoint = "Test3", CallingConvention = CallingConvention.Cdecl)]
        public static extern IntPtr Test3(string sour);

        [DllImport("Cpp.dll", EntryPoint = "Test4", CallingConvention = CallingConvention.Cdecl)]
        public static extern IntPtr Test4( );

        [DllImport("Cpp.dll", EntryPoint = "Test5", CallingConvention = CallingConvention.Cdecl)]
        public static extern IntPtr Test5();

        [DllImport("Cpp.dll", EntryPoint = "Test6", CallingConvention = CallingConvention.Cdecl)]
        public static extern IntPtr Test6();

        static void Main(string[] args)
        {
            string strSour = "CS Call C++ Dll!";

            Byte[] bPara = new Byte[100];    //新建字节数组  

            IntPtr pRet = strcpyTest(ref bPara[0], strSour);
            string strGet = System.Text.Encoding.Default.GetString(bPara, 0, bPara.Length);    //将字节数组转换为字符串  
            string strRet = Marshal.PtrToStringAnsi(pRet);

            Console.WriteLine("源字符串：");
            Console.WriteLine(strSour);

            Console.WriteLine("传出值：");
            Console.WriteLine(strGet);

            Console.WriteLine("返回值：");
            Console.WriteLine(strRet);

            string strPwd = new string('a', 200);
            string strMemo = new string('b', 100);

            string a = "2011080001";
            string b = "中文";
            string c = "380000001:13871061222:10:000000";

            IntPtr k = Test2(a, b, 0, "", 1, c, ref strPwd, ref strMemo);
            IntPtr aa = k;
            int i= aa.ToInt32();

            Test3("abc");

            Test4();

            Test5();

            Test6();

        }
    }
}



```
