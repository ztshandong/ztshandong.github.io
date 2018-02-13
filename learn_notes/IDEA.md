# MAC
```sh
Class JavaLaunchHelper is implemented in both /Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/bin/java (0x10d19c4c0) and /Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/jre/lib/libinstrument.dylib (0x10ea194e0). One of the two will be used. Which one is undefined.
据说是个老bug


Help | Edit Custom Properties
idea.no.launcher=true


【Preferences】- 【Editor】-【General】-【Console】- 【Fold console lines that contain】
add:  Class JavaLaunchHelper is implemented in both
```

# 技巧 MacOS X 10.5+
```java
安装插件emacsIDEAs
keymap搜索emacsIDEAs
AceJumpWord ⌘+L之后输入p，当前页面所有p字母都会有个编号

"str".sout   相当于System.out.println("str");
100.fori   for循环
name.field //创建字段
user.return; //return user;
user.nn //if(user!=null)

F2 定位错误

⇧+⌘+F 查找
⇧+⌘+R 替换
Edit-ToggleCase ⇧+⌘+U 转换大小写
Edit-Find-SelectAll Occurences ^+⌘+G 选中所有相同项 

Navigate-Class ⌘+O 根据类名精确查找(有选项)
Navigate-File ⇧+⌘+O 根据文件名精确查找
Navigate-Symbol ⌥+⌘+O 根据方法名精确查找

⌘+1,7  在模块之间跳转
⌘+E 打开最近文件
⇧+⌘+E 最近修改的文件
Help-Find Action 

Navigate-Back 上一次光标位置 ⌘+[
Navigate-Next 下一次光标位置 ⌘+]
Navigate-Last Edit Location 上一次编辑位置 ⇧+⌘+L （自定义）
Navigate-Next Edit Location 下一次编辑位置 ^+⌘+L （自定义）

⌘+2打开Favorites项目卡

书签
⌥+F3 添加有编号的书签，之后可以用^+1，2来跳转书签
⌘+F3 显示所有书签

⌥+⇧+F 添加到收藏（光标在类名上就收藏类，光标在方法名上就收藏方法）

Live Templates-Add Live Template Group，Add Live Template

/**
 * @author $USER$ on $DATE$.
 */
public static void testMethod(){
    int $var2$;
    $var2$=$var3$;
    $END$
}
Edit variables-USER Expression user(),DATE Expression date()

File and Code Templates - Includes -File Header
Application in 选择java

Postfix Completion 内置简写

Refactor-Extract-Variable 抽取变量

```