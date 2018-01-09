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


