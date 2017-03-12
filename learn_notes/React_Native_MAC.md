# 安装Homebrew
```sh
 /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
 在Max OS X 10.11（El Capitan)版本中，homebrew在安装软件时可能会碰到/usr/local目录不可写的权限问题，
 需执行sudo chown -R `whoami` /usr/local
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
都是属于这个flow工具的语法。这一语法并不属于ES标准，只是Facebook自家的代码规范。所以新手可以直接跳过
（即不需要安装这一工具，也不建议去费力学习flow相关语法）。
brew install flow
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
Android Studio需要Java Development Kit [JDK] 1.8或更高版本。你可以在命令行中输入 javac -version










