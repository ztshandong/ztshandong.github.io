### sudo spctl --master-disable
### sudo su -
```sh
cp pushall.cmd pushall.sh
chmod +x pushall.sh
```
# 安装Homebrew
```sh
 /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
 在Max OS X 10.11（El Capitan)版本中，homebrew在安装软件时可能会碰到/usr/local目录不可写的权限问题，
 需执行sudo chown -R `whoami` /usr/local
 ```
# Homebrew提速
```sh
一：中科大
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
二：清华
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-science"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-science.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-python"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-python.git
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
三：重置
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git

brew update  更新brew
brew upgrade 更新软件
```
# 安装vscode
brew cask install visual-studio-code

# 安装环境
```java
brew search tomcat
brew install tomcat@7
echo 'export PATH="/usr/local/opt/tomcat@7/bin:$PATH"' >> ~/.bash_profile

brew install tomcat-native
vi $CATALINA_HOME/bin/setenv.sh
添加
CATALINA_OPTS="$CATALINA_OPTS -Djava.library.path=/usr/local/opt/tomcat-native/lib"

apr
echo 'export PATH="/usr/local/opt/apr/bin:$PATH"' >> ~/.bash_profile

java
export PATH="/usr/local/opt/tomcat@7/bin:$PATH;.;$PATH:$JAVA_HOME/bin"
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home
export CLASS_PATH="$JAVA_HOME/lib"

eclipse的java api项目主要更新build下的文件就可以，注意某些配置文件不用更新
```
