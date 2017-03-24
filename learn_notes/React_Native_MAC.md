# 安装Xcode
```sh
安装完后执行
xcode-select --install
安装完重启
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
替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
二：清华
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
三：重置
重置brew.git:
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
重置homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git

brew update  更新brew
brew upgrade 更新软件
```
# 安装Node
```sh
brew install node
```
### 安装完node后建议设置npm镜像以加速后面的过程（或使用科学上网工具）。
### 注意：不要使用cnpm！cnpm安装的模块路径比较奇怪，packager不能正常识别！
```sh
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```

# Yarn、React Native的命令行工具（react-native-cli）
```sh
npm install -g yarn react-native-cli
yarn config set registry https://registry.npm.taobao.org --global
yarn config set disturl https://npm.taobao.org/dist --global
```
# 升级
```sh
npm info react-native 查看最新版本号
打开项目目录下的package.json文件，然后在dependencies模块下找到react-native，将当前版本号改到最新
sudo npm install
react-native upgrade
```
# MAC
# Watchman
#### Watchman是由Facebook提供的监视文件系统变更的工具。
#### 安装此工具可以提高开发时的性能（packager可以快速捕捉文件的变化从而实现实时刷新）。
```sh
brew install watchman
```
### Flow
```sh
Flow是一个静态的JS类型检查工具。译注：你在很多示例中看到的奇奇怪怪的冒号问号，以及方法参数中像类型一样的写法，
都是属于这个flow工具的语法。这一语法并不属于ES标准，只是Facebook自家的代码规范。
brew install flow
```
# 安装常用开发工具
```sh
brew install git gcc pkg-config cairo libpng jpeg gitlib 
```
# Nuclide
#### Nuclide（此链接需要科学上网）是由Facebook提供的基于atom的集成开发环境，
#### 可用于编写、运行和 调试React Native应用。
#### 译注：我们更推荐使用WebStorm或Sublime Text来编写React Native应用。
```sh
react-native init AwesomeProject
cd AwesomeProject
react-native run-ios
你也可以在Nuclide中打开AwesomeProject文件夹 然后运行，
或是双击ios/AwesomeProject.xcodeproj文件然后在Xcode中点击Run按钮。
```

# Android
- Android Studio需要Java Development Kit [JDK] 1.8或更高版本
```sh
javac -version
```
### [下载JDK](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)
### 下载离线sdk[android-sdk-24.4.1_1](https://homebrew.bintray.com/bottles/android-sdk-24.4.1_1.el_capitan.bottle.tar.gz)
###### [原文链接](https://gist.github.com/Erichain/0ac3a6aaca0c28ad6551)
### 下载路径中打开终端后执行
```sh
$ cp android-sdk-24.4.1_1.el_capitan.bottle.tar.gz $(brew --cache android-sdk)
解压到brew缓存文件夹中后执行
$ brew install android-sdk
此时安装路径为/usr/local/opt/android-sdk
如果通过Android Studio安装的sdk，则其路径为export ANDROID_HOME=~/Library/Android/sdk
brew install gradle 如果安装Android Studio则不必安装gradle
```
### 配置变量
```sh
vi ~/.bash_profile
添加
export ANDROID_HOME="/usr/local/opt/android-sdk"
export GRADLE_HOME=/usr/local/opt/gradle
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
:wq
然后使用下列命令使其立即生效（否则重启后才生效）：
source ~/.bash_profile
echo $ANDROID_HOME检查此变量是否已正确设置。
```
### 下载开发版本包
```sh
在SDK Tools窗口中，选择Show Package Details，然后在Android SDK Build Tools中
勾选Android SDK Build-Tools 23.0.1（必须是这个版本）。
然后还要勾选最底部的Android Support Repository.

在SDK Platforms窗口中，选择Show Package Details，然后在Android 6.0 (Marshmallow)中
勾选Google APIs、Android SDK Platform 23、Intel x86 Atom System Image、
Intel x86 Atom_64 System Image以及Google APIs Intel x86 Atom_64 System Image。
```
### 使用Android Studio创建AVD
```sh
如果是vmware，则需要勾选 Intel VT-x/EPT 与 AMD-RVI ，否则MAC虚拟机中无法开启安卓虚拟机
然后运行查看命令
adb devices
第一列表示设备 ID，第二列表示设备状态，device 表明可以运行。
```
### 下载[gradle-2.14.1-all.zip](https://services.gradle.org/distributions/gradle-2.14.1-all.zip)
```sh
Android Studio打开一个AVD后运行
react-native run-android
此时会下载N多jar包
如果下载gradle-2.14.1-all.zip慢就查看路径
/Users/zhangtao/.gradle/wrapper/dists/gradle-2.14.1-all/8bnwg5hd3w55iofp58khbp6yv/gradle-2.14.1-all.zip
```
### 真机调试
```sh
点击屏幕左上角苹果标志->关于本机->更多信息->系统报告，在左侧列表选择 USB，就能看到对应的 USB 设备厂商号。
小米1的一般是 0x18dl，小米 2 以后 和 红米应该是 0x2717。
echo "0x2717" >> ~/.android/adb_usb.ini
然后重启adb
adb kill-server
adb start-server
```
###### 几个网站
[电子科技大学](http://mirrors.dormforce.NET)

[东软信息学院](http://mirrors.neusoft.edu.cn) 

[北京化工大学](http://ubuntu.buct.edu.cn/ubuntu.buct.cn) 

[中国科学院开源协会1](http://mirrors.opencas.cn) 

[中国科学院开源协会2](http://mirrors.opencas.org)

[中国科学院开源协会3](http://mirrors.opencas.ac.cn)

[上海GDG镜像服务器](http://sdk.gdgshanghai.com:8000)



