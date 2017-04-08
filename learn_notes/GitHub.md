
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
- 第一步，先用 ssh-kengen 公钥私钥对，如果多个网站是用同一个邮箱注册的就不用分
```unix
ssh-keygen -t rsa -C "email@gmail.com" -f ~/.ssh/git_rsa
windows系统要写成
ssh-keygen -t rsa -C "email@gmail.com" -f c:\users\GitRSA\.ssh\git_rsa   装系统时用户名最好不要用中文
-f表示路径,git_rsa是文件名
```
- 如果邮箱不同要分别保存
```sh
$ ssh-keygen -t rsa -C "xxx@github.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_github

$ ssh-keygen -t rsa -C "xxx@oschina.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_gitosc

$ ssh-keygen -t rsa -C "xxx@gitlab.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bao/.ssh/id_rsa): id_rsa_gitlab
```
- 将公钥添加到对应的网站
```sh
cat git_rsa.pub
```
- 第二步，使用 ssh-add 命令将新的 ssh 私钥添加到 ssh agent 中，因为默认只识别 id_rsa。
```unix
$ ssh-agent bash
$ ssh-add c:\users\GitRSA\.ssh\git_rsa

$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_ras_gitosc
$ ssh-add ~/.ssh/id_ras_gitlab
```
- 第三步，配置 ~/.ssh/config 文件，如果此文件不存在，则新建一个。
- touch config
```unix
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

使用的格式为ssh -vT github   这个格式其实不方便，clone的时候要改
Host github                          // 这个名字随便取，用来取代ssh地址中的 git@github.com
  HostName github.com                // @ 与 : 之间的内容
  User git                           // @ 之前的内容
  IdentityFile ~/.ssh/id_rsa_github  // 对应的私钥文件
```
- 第四步，连接
```unix
$ ssh -vT git@github.com              
$ ssh -vT github        使用别名 

$ ssh -vT git@git.oschina.net 
$ ssh -vT git@gitlab.com

$ git clone git@github.com:name/projectname.github.io.git   
```
- 第五步，在每个本地项目中添加ssh，如果是private就用OSC与GitLab，public就再添加GitHub，
```sh
git remote rm origin
git remote add osc "OSC仓库的ssh格式地址"
git push --set-upstream osc master
git remote add gitlab "GitLab仓库的ssh格式地址"
git push --set-upstream gitlab master
git remote add github "Git仓库的ssh格式地址"
git push --set-upstream github master

git add .
git commit -am 'Description'

可以创建个脚本或者批处理
pushall.cmd   添加
git push github master
git push osc master
git push gitlab master
以后运行./pushall.cmd即可同步多个
```






# 下面的不用看了
```unix
cd  ~/.ssh
ls




生成sshkey


ssh-keygen -t ras -C "youremail@yourcompany.com"
这时可以一路回车，不输入任何字符，将自动生成id_rsa和id_rsa.pub文件。

生成并添加第二个ssh key
ssh-keygen -t rsa -C "youremail@gmail.com"

 ： 这时不能一路回车，否则邮箱将覆盖上一次生成的ssh key，给这个文件起一个名字， 
比如叫 id_rsa_github,所以相应的也会生成一个 id_rsa_github.pub 文件。

添加ssh key
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/id_rsa_github

在 ~/.ssh 目录下新建一个config文件

touch config

# gitlab
Host gitlab.com
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
测试
$ ssh -T git@github.com



# 多github账号[原文](http://www.cnblogs.com/BeginMan/p/3548139.html)
# 新建SSH key：
$ cd ~/.ssh     # 切换到C:\Users\Administrator\.ssh
ssh-keygen -t rsa -C "mywork@email.com"  # 新建工作的SSH key
# 设置名称为id_rsa_work
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa): id_rsa_work
因为默认只读取id_rsa，为了让SSH识别新的私钥，需将其添加到SSH agent中：
ssh-add ~/.ssh/id_rsa_work
如果出现Could not open a connection to your authentication agent的错误，就试着用以下命令：
ssh-agent bash
ssh-add ~/.ssh/id_rsa_work
在~/.ssh目录下找到config文件，如果没有就创建：
touch config
# 该文件用于配置私钥对应的服务器
# Default github user(first@mail.com)
Host github.com
 HostName github.com
 User git
 IdentityFile C:/Users/Administrator/.ssh/id_rsa

 # second user(second@mail.com)
 # 建一个github别名，新建的帐号使用这个别名做克隆和更新
Host github2
 HostName github.com
 User git
 IdentityFile C:/Users/Administrator/.ssh/id_rsa_work
 
 打开新生成的~/.ssh/id_rsa2.pub文件，将里面的内容添加到GitHub后台。

可不要忘了添加到你的另一个github帐号下的SSH Key中。
ssh -T git@github.com
ssh -T github2

```
