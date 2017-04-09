
### HTTPS方式, push 大文件可能引发错误
### SSH方式，使用加密通道读写仓库，无单次上传限制
```sh
管理员power shell
C:\Users\username\AppData\Local\GitHub> ./GitHub.appref-ms --open-shell
```
# 处理中文乱码
```sh
ssh-agent bash
$ git config --global core.quotepath false  		# 显示 status 编码
$ git config --global gui.encoding utf-8			# 图形界面编码
$ git config --global i18n.commit.encoding utf-8	# 提交信息编码
$ git config --global i18n.logoutputencoding utf-8	# 输出 log 编码
$ export LESSCHARSET=utf-8
```
### 第一步，先用ssh-kengen产生公钥私钥对，如果多个网站是用同一个邮箱注册的就不用分
```sh
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
```
### 将公钥添加到对应的网站
```sh
cat git_rsa.pub
```
### 第二步，使用 ssh-add 命令将新的 ssh 私钥添加到 ssh agent 中，因为默认只识别 id_rsa。
```sh
$ ssh-agent bash
$ ssh-add c:\users\GitRSA\.ssh\git_rsa

$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_ras_gitosc
$ ssh-add ~/.ssh/id_ras_gitlab
$ ssh-add ~/.ssh/id_ras_bitbucket
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

使用的格式为ssh -vT github   这个格式其实不方便，clone的时候要改
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
ssh -vT git@bitbucket.org

$ ssh -vT github        使用别名 
$ git clone git@github.com:name/projectname.github.io.git   
```
### 第五步，在每个本地项目中添加ssh，如果是private就用OSC,GitLab,BitBucket
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

git add .
git commit -am 'Description'

可以创建个脚本或者批处理
pushall.cmd   添加
git push osc master
git push gitlab master
git push bitbucket master
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
删除分支：$ git branch -d [slave] ---- -d选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用-D选项

合并分支：$ git merge --no-ff [slave] ----将名称为[slave]的分支与当前分支合并
合并分支时首先切换到master，然后将其他可以用的分支merge提交
然后切换到其他分支下载最新的master


创建远程分支(本地分支push到远程)：
git push osc [slave]
git push gitlab [slave]
git push bitbucket [slave]
git push github [slave]   public
删除远程分支：$ git push origin :heads/[slave] 或 $ git push origin :[slave] 

git fetch osc slave  取回osc的slave分支
```
[git中merge与rebase区别](http://www.cnblogs.com/xueweihan/p/5743327.html)