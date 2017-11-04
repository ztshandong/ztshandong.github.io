# 回滚
```sh
回滚使用的是power tools中的tfpt rollack命令。tfpt需要运行在powershell中。

具体步骤：
1.先在Team Explorer中将要回滚的工作区映射到本地
2.打开开始菜单中power tools里的powershell
3.使用cd命令导航到你映射到的目录，例如cd c:\project1 （假如你将项目映射到c:\project1目录）
4.输入tfpt rollback，它会提示你是否获取最新版本，选YES
5.选择要回滚的变更集(注意：这个是你要撤销操作的变更集)
6.回滚之后，还必须执行签入操作，回滚在被提交到服务器。

注意事项：
执行rollback的时候必须保证所有本工作区中的项目没有挂起更改，不单单是你要回滚的目录下的内容没有挂起哦。否则，你就会收到如下信息：
Cannot proceed because you have pending changes in your workspace. You must move
 to a shelveset, undo, or check in all pending changes before reverting a change
set.
```