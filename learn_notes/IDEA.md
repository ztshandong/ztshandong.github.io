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

+⌘+V 调出剪贴板

Call Hierarchy ^+⌥+H 查看所有引用

⌘+1,7  在模块之间跳转
⌘+E 打开最近文件
⇧+⌘+E 最近修改的文件
Help-Find Action 

Navigate-Back 上一次光标位置 ⌘+[
Navigate-Next 下一次光标位置 ⌘+]
Navigate-Last Edit Location 上一次编辑位置 ⇧+⌘+L （自定义）
Navigate-Next Edit Location 下一次编辑位置 ^+⌘+L （自定义）

Navigate-File Structure ⌘+F12 类大纲

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

# 快捷键模式为Visual Studio
```
 IDEA快捷键模式为Visual Studio
 Setting - Editor - File and Code Templates - Class 模板
 File Header
 /**
* @author:zhangsan
* @date:${YEAR}/${MONTH}/${DATE}/${TIME}
* @remarke:
*/
 Setting - Editor - Live Templates Custrom
 ft field template
(Edit Template Variables)
DATE与TIME选skip if defined
date()，time()
 /**
 * @author:zhangsan
 * @date:$DATE$/$TIME$
 * @description:$var1$ 
 */
private $var2$ $var3$;

mt method template
/**
 * @author:zhangsan
 * @date:$DATE$/$TIME$
 * @description:$var1$
 * @input:$var2$
 * @output:$var3$
 */
public $var4$ $var5$(){
    
}
 Settings - Keymap - Find in Path 改成Ctrl + Shift + Q

 数字.fori + TAB 增续循环
 数字.forr + TAB 降序循环
 sout + TAB
 const + TAB 生成final static
 iter + TAB 上面数下来最近的一个list进行foreach
 itli + TAB list迭代
 itco + TAB iterator迭代
 inst + TAB xxx instanceof ooo
 souf + TAB System.out.printf("");

 F2 下一个错
 F4 跳到定义处
 Shift*2 搜索

 Alt + 1 打开Project视图
 Alt + 2 查看书签
 Alt + 6 查看TODO
 Alt + 7 显示当前类结构

 Alt + F9 显示所有断点
 Alt + F12 显示terminal

 Alt + 上箭头 上一个方法
 Alt + HOME 定位到导航条
 Alt + Left 切换选项卡
 Alt + Enter 自动导入包
 Alt + Insert 生成getter setter
 Alt + J 依次往下选择相同内容
 Alt + Shift + J 从下往上依次取消选择

 Ctrl + F 查找
 Ctrl + H 替换
 Ctrl + X 删除行
 Ctrl + D 复制行
 Ctrl + E 最近打开的文件

 Shift + F6 重命名
 Shift + Enter 另起一行
 Shift + HOME/END 从当前位置选择到本行的头/尾

 Ctrl + -/+ 折叠/展开
 Ctrl + W 扩展选择内容
 Ctrl + TAB 切换选项卡
 Ctrl + [] 定位到代码块开始/结束处

 Ctrl + Alt + F 格式化
 Ctrl + Alt + O 去除未使用引用
 Ctrl + Alt + I 自动格式化当前行
 Ctrl + Alt + left 回到上次位置
 Ctrl + Alt + F12 打开文件夹
 Ctrl + Alt + Shift + T 重构

 Ctrl + Shift + O override
 Ctrl + Shift + I 实现接口
 Ctrl + Shift + U 切换大小写
 Ctrl + Shift + J 两行合并一行
 Ctrl + Shift + E 最近编辑的文件
 Ctrl + Shift + 上下键 移动代码
 Ctrl + Shift + N 搜索文件
 Ctrl + Shift + Z 取消撤销
 Ctrl + Shift + C 复制绝对路径
 Ctrl + Shift + V 显示剪贴板
 Ctrl + Shift + I 实现接口
 Ctrl + Shift + R 全局替换
 Ctrl + Shift + M 显示对应的{}
 Ctrl + Shift + Alt + C 复制相对路径

 Ctrl + Alt + F7 弹窗显示所有引用
 Ctrl + Alt + F12 弹出路径框

 Ctrl + Alt + Shift + J 选择所有一样的内容
 Ctrl + Alt + Shift + F7 下方显示所有引用

 Ctrl/Alt + Shift + 上下箭头 代码上移下移

```
