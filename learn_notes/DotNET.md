# 程序集混淆签名
```sh
vs2015开发人员工具，管理员权限运行
sn -k cppkey.snk
sn -k csharp.snk
sn -p csharp.public.snk

c++项目调用一次snk就会使snk发生变化，所以c++项目与c#项目绝对不要使用同一个密钥，否则会签名失败
在cpp文件构造函数上面添加三行
using namespace System::Reflection;
[assembly:AssemblyKeyFileAttribute("cppkey.snk")];
[assembly:AssemblyDelaySignAttribute(true)];
项目属性-生成事件-预先生成事件-命令行：sn -k $(OutDir)\cppkey.snk

生成cpp.dll
发布时使用Dotfuscator混淆时可同时签名，否则就执行
sn -Ra cpp.dll cppkey.snk
然后就可以给其他项目使用了


c#项目属性中选择为程序集签名与延迟签名，选择csharp.public.snk
延迟签名后程序不能直接运行，需要使用-Vr免验证Hash（但是直接混淆后貌似可以运行）
sn -Vr test.dll
sn -Vr test.exe
发布时使用Dotfuscator混淆时可同时签名，否则就执行
sn -Ra test.dll csharp.snk
sn -Ra test.exe csharp.snk

csharp.public.snk是提取出来的公钥，给开发人员使用。
csharp.snk是公私钥对，要由专人保管。
```