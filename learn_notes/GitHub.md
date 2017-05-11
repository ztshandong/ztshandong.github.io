
### HTTPS方式, push 大文件可能引发错误
### SSH方式，使用加密通道读写仓库，无单次上传限制
```sh
管理员power shell
C:\Users\username\AppData\Local\GitHub> ./GitHub.appref-ms --open-shell
```
# 处理中文乱码
```javascript
ssh-agent bash
$ git config --global core.quotepath false  		# 显示 status 编码
$ git config --global gui.encoding utf-8			# 图形界面编码
$ git config --global i18n.commit.encoding utf-8	# 提交信息编码
$ git config --global i18n.logoutputencoding utf-8	# 输出 log 编码
$ export LESSCHARSET=utf-8
```
### 第一步，先用ssh-kengen产生公钥私钥对，如果多个网站是用同一个邮箱注册的就不用分
```javascript
ssh-keygen -t rsa -C "email@gmail.com" -f ~/.ssh/git_rsa
windows系统要写成
ssh-keygen -t rsa -C "email@gmail.com" -f c:\users\GitRSA\.ssh\git_rsa   装系统时用户名最好不要用中文
-f表示路径,git_rsa是文件名
```
### 如果邮箱不同可能要分别保存，未验证
```sh
$ ssh-keygen -t rsa -C "xxx@github.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_github

$ ssh-keygen -t rsa -C "xxx@oschina.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_gitosc

$ ssh-keygen -t rsa -C "xxx@bitbucket.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_bitbucket

$ ssh-keygen -t rsa -C "xxx@gitlab.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_gitlab

$ ssh-keygen -t rsa -C "xxx@aliyun.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_aliyun
```
### 将公钥添加到对应的网站
```javascript
cd C:\Users\GitRSA\.ssh
cat git_rsa.pub
```
### 第二步，使用 ssh-add 命令将新的 ssh 私钥添加到 ssh agent 中，因为默认只识别 id_rsa。
```sh
$ ssh-agent bash
$ ssh-add c:\users\GitRSA\.ssh\git_rsa
如果多个证书要添加多次
$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_ras_gitosc
$ ssh-add ~/.ssh/id_ras_gitlab
$ ssh-add ~/.ssh/id_ras_bitbucket
$ ssh-add ~/.ssh/id_ras_aliyun
```
### 第三步，配置 ~/.ssh/config 文件，如果此文件不存在，则新建一个。
### touch config
```sh
使用的格式为ssh -vT git@github.com  推荐
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile c:\users\GitRSA\.ssh\git_rsa
 Host git.oschina.net
    HostName git.oschina.net
    PreferredAuthentications publickey
    IdentityFile c:\users\GitRSA\.ssh\git_rsa 
 Host gitlab.com
    HostName www.gitlab.com
    PreferredAuthentications publickey
    IdentityFile c:\users\GitRSA\.ssh\git_rsa 
 Host bitbucket.org
    HostName bitbucket.org
    PreferredAuthentications publickey
    IdentityFile c:\users\GitRSA\.ssh\git_rsa
Host code.aliyun.com
    HostName code.aliyun.com
    IdentityFile c:\users\GitRSA\.ssh\git_rsa

mac要改为IdentityFile ～／.ssh／git_rsa

别名使用的格式为ssh -vT github   这个格式其实不方便，clone的时候要改
Host github                          // 这个名字随便取，用来取代ssh地址中的 git@github.com
  HostName github.com                // @ 与 : 之间的内容
  User git                           // @ 之前的内容
  IdentityFile ~/.ssh/id_rsa_github  // 对应的私钥文件
```
### 第四步，连接
```sh
$ ssh -vT git@github.com    
$ ssh -vT git@git.oschina.net 
$ ssh -vT git@gitlab.com
$ ssh -vT git@bitbucket.org
$ ssh -vT git@code.aliyun.com

mac
bad owner or permissions on .ssh/config
chmod 600 ~/.ssh/config
Permissions 0777 for '.ssh/git_rsa' are too open
chmod 400 ~/.ssh/git_rsa

$ ssh -vT github        使用别名 
$ git clone git@github.com:name/projectname.github.io.git   
```
### 第五步，在每个本地项目中添加ssh，如果是private就用OSC,GitLab,BitBucket,Aliyun
### public就再添加GitHub，
```sh
git remote rm origin
git remote add osc "OSC仓库的ssh格式地址"
git push --set-upstream osc master
git remote add gitlab "GitLab仓库的ssh格式地址"
git push --set-upstream gitlab master
git remote add github "Git仓库的ssh格式地址"
git push --set-upstream github master
git remote add bitbucket "bitbucket仓库的ssh格式地址"
git push --set-upstream bitbucket master
git remote add aliyun "aliyun仓库的ssh格式地址"
git push --set-upstream aliyun master

git add .
mac commit 之前要设置
git config --global user.name "Your Name"
git config --global user.email you@example.com

After doing this, you may fix the identity used for this commit with:
git commit --amend --reset-author

git commit -am 'Description'

commit之后要merge然后push
合并本地分支：$ git merge --no-ff [slave] ----将名称为[slave]的分支与当前分支合并
masterPush.cmd
//远程分支合并
git push osc master:slaveBranch  // 提交本地master分支作为远程的slaveBranch分支
git checkout slaveBranch
git pull osc slaveBranch  pull相当于fetch后merge
如果有多个源，pull之后可能会显示与其中某个不同步，例如显示与aliyun不一样，再pull一下aliyun就可以

git fetch osc slaveBranch  取回osc的slaveBranch分支
git merge osc/slaveBranch

git diff master origin/master
可以创建个脚本或者批处理
pushall.cmd   添加
git push osc master
git push gitlab master
git push bitbucket master
git push aliyun master
git push github master   public
以后运行./pushall.cmd即可同步多个
```
### 常用命令
```sh
查看本地分支：$ git branch
查看所有分支：$ git branch -a
查看远程分支：$ git branch -r
创建本地分支：$ git branch [slave] ----注意新分支创建后不会自动切换为当前分支
切换分支：$ git checkout [slave]
创建新分支并立即切换到新分支：$ git checkout -b [slave]
slavePush.cmd
删除本地分支：$ git branch -d [slave] ---- -d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项


创建远程分支(本地分支push到远程)：
git push osc [slave]
git push gitlab [slave]
git push bitbucket [slave]
git push aliyun [slave]
git push github [slave]   public
删除远程分支：git push osc --delete slaveBranch

```
# 冲突
### 冲突的产生
```java
merge时冲突的机率最大，pull时和merge分支时都会有冲突
rebase是重新设置基准，然后patch时也会冲突
git pull会自动merge，repo sync会自动rebase，所以git pull和repo sync也会产生冲突。当然git rebase就更不用说了。
```
### 冲突类型
##### 树冲突
```java
树冲突
文件名修改造成的冲突，称为树冲突。
比如，a用户把文件改名为a.c，b用户把同一个文件改名为b.c，那么b将这两个commit合并时，会产生冲突。
$ git status
    added by us:    b.c
    both deleted:   origin-name.c
    added by them:  a.c
如果最终确定用b.c，那么解决办法如下：
git rm a.c
git rm origin-name.c
git add b.c
git commit
执行前面两个git rm时，会告警“file-name : needs merge”，可以不必理会。


关于树冲突，出现的原因是因为同时对一个文件进行了重命名。也可以使用mergetool修复冲突，但是更直接的方法是直接使用git rm删除想删除的文件，使用git add将需要的文件加到暂存区进而commit。

git ls-files –s
可查看暂存区文件。

树冲突也可以用git mergetool来解决，但整个解决过程是在交互式问答中完成的，用d 删除不要的文件，用c保留需要的文件。
最后执行git commit提交即可。
```

##### 逻辑冲突
```java
git自动处理（合并/应用补丁）成功，但是逻辑上是有问题的。
比如另外一个人修改了文件名，但我还使用老的文件名，这种情况下自动处理是能成功的，但实际上是有问题的。
又比如，函数返回值含义变化，但我还使用老的含义，这种情况自动处理成功，但可能隐藏着重大BUG。这种问题，主要通过自动化测试来保障。所以最好是能够写出比较完备的自动化测试用例。
这种冲突的解决，就是做一次BUG修正。不是真正解决git报告的冲突。
```

##### 内容冲突
```java
两个用户修改了同一个文件的同一块区域，git会报告内容冲突。一般来讲，出现冲突时都会有“CONFLICT”字样：
$ git pull
Auto-merging test.txt
CONFLICT (content): Merge conflict in test.txt
Automatic merge failed; fix conflicts and then commit the result.

但是，也有例外，repo sync的报错，可能并不是直接提示冲突，而是下面这样：
error: project mini/sample
无论是否存在冲突，只要本地修改不是基于服务器最新的，它都可能报告这个错误，解决方法都是一样。
 
这个时候，需要进入报错的项目（git库）目录，然后执行git rebase解决：
git rebase remote-branch-name
```
### 解决冲突
```java
merge/patch的冲突解决

先编辑冲突，然后git commit提交。
注：对于git来讲，编辑冲突跟平时的修改代码没什么差异。修改完成后，都是要把修改添加到缓存，然后commit。
rebase的冲突解决

rebase的冲突解决过程，就是解决每个应用补丁冲突的过程。
解决完一个补丁应用的冲突后，执行下面命令标记冲突已解决（也就是把修改内容加入缓存）：
git add -u
-u 表示把所有已track的文件的新的修改加入缓存，但不加入新的文件。
 
然后执行下面命令继续rebase：
git rebase --continue
有冲突继续解决，重复这这些步骤，直到rebase完成。
 
如果中间遇到某个补丁不需要应用，可以用下面命令忽略：
git rebase --skip
如果想回到rebase执行之前的状态，可以执行：
git rebase --abort
rebase之后，不需要执行commit，也不存在新的修改需要提交，都是git自动完成。
```
### 编辑冲突
```java
冲突文件里会有冲突标记
冲突标记<<<<<<< （7个<）与=======之间的内容是我的修改，=======与>>>>>>>之间的内容是别人的修改。
最简单的编辑冲突的办法，就是直接编辑冲突了的文件（test.txt），把冲突标记删掉，把冲突解决正确。
如果要解决的冲突很多，且比较复杂，图形界面的冲突解决工具就显得很重要了。
 
执行git mergetool用预先配置的Beyond Compare解决冲突：
启动Beyond Compare之后，会自动生成几个包含大写字母名称、数字的辅助文件：
关闭Beyond Compare时，这几个辅助文件都会自动删除，但同时会生成一个test.txt.orig的文件，内容是解决冲突前的冲突现场。
默认该.orig文件可能不会自动删除，需要手动删掉。
```
[git中merge与rebase区别](http://www.cnblogs.com/xueweihan/p/5743327.html)