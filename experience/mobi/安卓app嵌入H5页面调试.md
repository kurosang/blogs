# 安卓app嵌入H5页面调试

## 安装流程（MAC）
1. 科学上网
2. 安装homeBrew

    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

3. 更换Homebrew 镜像

    https://mirror.tuna.tsinghua.edu.cn/help/homebrew/

    参照上面的网址更新，坑：第三个可能会没有该文件夹，先创建该文件夹
4. 安装android-platform-tools

    注意！这个写法 brew cask install android-platform-tools **已经过时**！！！

    brew 可以装，移到 cask 了

    brew install Caskroom/cask/android-platform-tools

## 使用方法

首先USB连接手机和电脑，其次，打开手机上的USB调试模式。

在命令行终端上运行
主要命令：

* adb kill-server
* adb start-server
* adb devices

