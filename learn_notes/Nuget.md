# 创建可同时用于.net core与.net framework的库
```c#
vs2017
新建-类库(.NET Standard)
编辑csproj文件TagetFramework注意添加一个s
<TagetFrameworks>net45;netstandard2.0</TagetFrameworks>
例如获取程序工作文件夹路径的方法
使用.net framework4.5的方法是
string rootDir = AppDomain.CurrentDomain.BaseDirectory
使用.net core
 string rootDir = AppContext.BaseDirectory;

 对于这样有差异的代码我们应该使用条件编译的方法兼容,方法如下
查看项目的编译符号,项目->右键->属性->生成

可以看到项目的生成符号是NET45,我们的兼容代码就可以这样编写
public class Class1
{
#if NET45
    string rootDir = AppDomain.CurrentDomain.BaseDirectory
#else
    string rootDir = AppContext.BaseDirectory;
#endif
}
并且可以在导航栏来切换不同框架版本来进行调试
项目右键-打包或者发布会生成nupkg文件
```