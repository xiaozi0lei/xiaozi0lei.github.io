---
layout: post
title:  "win7 安装 Appium"
date:   2017-02-16
categories: appium
---

**1. Appium**
Appium是CS架构设计，通过client端发送RESTful API命令给server，server转换成对应测试平台的指令，触发操作，同时收集操作后的返回结果，响应给client端。
从官方下载的AppiumForWindows.zip包安装后，是自带node.exe的，不需要单独下载nodejs。

**2. Appium.app和Appium.exe**
appium server的GUI版本，前者用在osx上，后者是windows上。可视化、不需要装node，可以看app的UI结构是这个东东的卖点。

* 下载安装Appium.exe
AppiumForWindows_1_4_13_1.zip http://pan.baidu.com/s/1c0t28vQ
* 下载安装JDK
* 下载安装Android SDK Manager
installer_r24.3.4-windows.exe http://pan.baidu.com/s/1c0t28vQ
* 下载对应的Android SDK
启动Android SDK Manager，勾选对应的SDK下载


**3. Appium client**
* 安装ruby
windows上下载rubyinstaller安装ruby，选择2.17以上的版本。 http://rubyinstaller.org/downloads/
* 安装ruby client
先将gem source换成淘宝的源
```bash
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
gem sources --list
```
更新rubygems和bundler
```bash
gem update --system
gem install --no-rdoc --no-ri bundler 
```
安装最新的Appium client gem
~~安装过程会提示缺少build tools，先下载安装DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe http://pan.baidu.com/s/1c0t28vQ~~
安装appium_lib
```bash
gem uninstall -aIx appium_lib
gem install --no-rdoc --no-ri appium_lib
```
安装appium_console
`gem install --no-rdoc --no-ri appium_console bond`

**4. 启动Appium server**
* 运行appium.exe启动Appium server
* 创建appium.txt文件
```bash
[caps]
platformName = "android"
app = "./api.apk"
```
* 在cmd窗口中切换到appium.txt文件所在目录，执行`arc`命令
这样便进去了pry命令行，可以练习各种命令。

___

下载：
http://appium.io/
https://bitbucket.org/appium/appium.app/downloads/
教程：
http://appium.io/slate/en/master/
http://appium.io/slate/en/tutorial/android.html
http://www.cnblogs.com/nbkhic/tag/appium/
https://github.com/appium/appium/tree/master/docs/cn
https://github.com/appium/appium/blob/master/docs/cn/appium-setup/running-on-windows.cn.md
登陆淘宝的NPM镜像网站，下载windows x64版本的nodejs安装包，安装后就有npm工具。

___

**依赖：**
淘宝NPM镜像
http://npm.taobao.org/

也可以源码安装
* 通过npm安装Appium server
```bash
npm install -g appium
# 上方的命令不成功，可以通过下方的命令尝试
npm -g --registry http://registry.cnpmjs.org  install appium
```
