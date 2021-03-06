---
layout: post
title: "使用 Ubuntu 编译 ijkplayer 源码"
excerpt: "基于版本 0.8.8"
date: 2018-4-18
categories:
  - ijkplayer
tags:
  - ijkplayer
---

-------------------

## 0x00 安装 Ubuntu
> 我用的是 [Oracle VM VirtualBox](https://www.virtualbox.org/) 虚拟机来安装 [Ubuntu 64位](https://www.ubuntu.com/download)，不会对已安装的系统造成什么影响。在新建的虚拟机时配置内存要选用大一点的，第一次我安装全是默认项，卡的要死，建议分配内存 4G，硬盘 30G 以上

-------------------

## 0x01 配置相关工具

### 1.配置 NDK

下载好的 NDK 解压 [NDK Download](https://dl.google.com/android/repository/android-ndk-r14b-linux-x86_64.zip)

![5](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 5.png)  

转到 Home 目录下，配置 NDK 路径，类似 Java 配置 JDK，很多教程 Android SDK 也要配置，其实是不用的
![6](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 6.png)  

键盘 Ctrl + H 显示隐藏文件，找到 .bashrc 文件

![7](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 7.png)  

将如下代码添加到文件末尾，保存并关闭，路径要写你自己的 NDK 路径
```
ANDROID_NDK=/home/qu/Downloads/android-ndk-r14b
export PATH=${PATH}:ANDROID_NDK
export ANDROID_NDK
```

你可以用命令来测试一下是否成功

```
adb -version
```

至此 NDK 配置结束了

### 2.编译前准备

安装一些软件

```
sudo apt-get update
sudo apt-get install git
sudo apt-get install yasm
```

下载 ijkplayer 项目

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
```

简单了解一下 ijkplayer 目录结构
![10](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 10.png)  

### 3.开始编译

进入 ijkplayer-android 目录
```
cd ijkplayer-android
```

如果你的代码不是最新的版本，最新版本号请查看 GitHub
```
git checkout -B latest k0.8.8
```

开始初始化
```
./init-android.sh
```

需要支持 Https 协议的需要执行如下命令
```
./init-android-openssl.sh
```

进入 contrib 目录
```
cd android/contrib
```

编译各个平台的编译 openssl
```
./compile-openssl.sh clean
./compile-openssl.sh all
```

编译 ffmpeg 解码库
```
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

返回上一目录
```
cd ..
```

获取 ijkplayer 的项目
```
./compile-ijk.sh all
```

至此所有编译结束了，你会在这个目录下看到如下文件

![8](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 8.png)  

最后复制出来，用 Android Studio 打开项目即可运行查看效果

![9](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 9.png)

-------------------

## 0x02 总结我遇到的一些问题

* ### 报错：不能为虚拟电脑  XXX 打开一个新任务
此时需要通过安装 [Oracle VM VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads) 扩展包解决此问题

![1](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 1.png)  

安装菜单目录 管理 → 全局设定 → 扩展

![2](/assets/image/2018-04-18/2018-04-18-Compile ijkplayer 2.png)  

* ### 请用 ndk r14b 版本
开始用的版本过低编译失败，又下了个最新的也失败，最后找到 r14b 版本，编译成功了

* ### 提示 You need the NDKr10e or later
检查 NDK 路径配置是否正确，或者 NDK 版本过高过低都会造成此问题

* ### 提示 ERROR:failed to create tool chain
通过安装 apt-get install python 解决此问题

* ### 安装下载时异常缓慢
推举一个免费 VPN 支持 Ubuntu 版的 [Lantern](https://github.com/getlantern/lantern)，使用如下命令安装

```
sudo apt install gdebi-core
sudo gdebi lantern.deb
```

-------------------

## 0x03 编译完成的项目地址

[https://github.com/RockyQu/ijkplayer](https://github.com/RockyQu/ijkplayer)