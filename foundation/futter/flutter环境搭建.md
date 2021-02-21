# flutter 环境搭建（Mac）

#### 安装 Flutter SDK

下载网址：https://flutter.dev/docs/development/tools/sdk/releases

选择最新稳定的版本（Stable 版本）

然后解压就可以了，我个人放在文稿目录里

#### 环境变量配置

添加全局路径，在根路径，比如我是在/Users/kuro 下找到`.bash_profile`打开它，在里面添加`export PATH=/Users/kuro/Documents/flutter/bin:$PATH`

**参考：（只需把路径的 kuro 换成你自己的用户名）**

如果没有科学上网，则还要加上

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

坑 1:如果没有`.bash_profile`文件，则自己创建就好。

#### 检查

在系统的终端输入 `flutter doctor`

坑 1: flutter 的命令失效， 提示 zsh: command not found: flutter

解决方法：https://blog.csdn.net/iotjin/article/details/105629266

成功之后然后出现

```
Last login: Mon Feb  1 14:31:42 on ttys002
wuhaomingdeMac-mini:~ kuro$ flutter doctor
Downloading Dart SDK from Flutter engine 2f0af3715217a0c2ada72c717d4ed9178d68f6ed...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  172M  100  172M    0     0  47.8M      0  0:00:03  0:00:03 --:--:-- 47.8M
Building flutter tool...
```

这个过程等挺久的

检查结果：

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 1.22.6, on Mac OS X 10.14.6 18G7016 darwin-x64, locale zh-Hans-CN)
[!] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from: https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK components.
      (or visit https://flutter.dev/docs/get-started/install/macos#android-setup for detailed
      instructions).
      If the Android SDK has been installed to a custom location, set ANDROID_HOME to that
      location.
      You may also want to add it to your PATH environment variable.

    ✗ No valid Android SDK platforms found in
      /usr/local/Caskroom/android-platform-tools/29.0.5/platforms. Directory was empty.
[✗] Xcode - develop for iOS and macOS
    ✗ Xcode installation is incomplete; a full installation is necessary for iOS development.
      Download at: https://developer.apple.com/xcode/download/
      Or install Xcode via the App Store.
      Once installed, run:
        sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
        sudo xcodebuild -runFirstLaunch
    ✗ CocoaPods not installed.
        CocoaPods is used to retrieve the iOS and macOS platform side's plugin code that
        responds to your plugin usage on the Dart side.
        Without CocoaPods, plugins will not work on iOS or macOS.
        For more info, see https://flutter.dev/platform-plugins
      To install:
        sudo gem install cocoapods
[!] Android Studio (not installed)
[!] VS Code (version 1.52.1)
    ✗ Flutter extension not installed; install from
      https://marketplace.visualstudio.com/items?itemName=Dart-Code.flutter

[!] Connected device
    ! No devices available

! Doctor found issues in 5 categories.

```

可以看到第一行，flutter 安装成功，下面安装其他工具

#### 安装 Android Studio

从 AS 官网直接下载 dmg 包进行安装，安装好之后运行，就会自动安装所需要的一些额外的东西。

之后运行 flutter doctor，我们还需要给 AS 安装 flutter 插件

流程：

```
启动Android Studio.
打开插件首选项 (Preferences>Plugins).
选择 Browse repositories…, 选择 Flutter 插件并点击 install.
重启Android Studio后插件生效.
```

当我们成功安装之后，重新运行 flutter doctor，发现还是提示我们没有安装，于是，找了一下原因，大概是：当 Flutter Doctor 无法找到 Android Studio 插件时，就会发生这种情况。这是 flutter 版本和 Android Studio 版本之间的问题。

但是，网上搜如果不用 Android Studio 来开发 flutter 是可以忽略的，但看到这个叉号，太不爽了。

使用下面办法，运行下面命令即可解决：

```
flutter channel dev
flutter doctor
flutter channel master
flutter doctor
```

其实本质就是切换了一下分支，推测有可能是因为没及时刷新？具体可点击下面

这个坑的具体解答：[点击了解](https://stackoverflow.com/questions/51860845/flutter-plugin-not-installed-error-when-running-flutter-doctor)

#### 第二点 Android toolchain - develop for Android devices

这一点也是打勾的，主要是在`.bash_profile`配置一些路径

参考：（只需把路径的 kuro 换成你自己的用户名）

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PATH=/Users/kuro/Documents/flutter/bin:$PATH
export ANDROID_HOME="/Users/kuro/Library/Android/sdk"
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
```

保存，退出终端重新运行 `doctor flutter`

发现还是有问题：

```
[!] Android toolchain - develop for Android devices (Android SDK 27.0.3)
    ✗ Android license status unknown.
```

从报错提示来看，需要添加 Android license。

执行命令：`flutter doctor --android-licenses`

一路输入 y，重新运行`flutter doctor`

```
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
```

完成

完成上面的步骤后，重新运行 `flutter doctor`

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel master, 1.26.0-18.0.pre.90, on Mac OS X 10.14.6 18G7016 darwin-x64, locale zh-Hans-CN)
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
[✗] Xcode - develop for iOS and macOS
    ✗ Xcode installation is incomplete; a full installation is necessary for iOS development.
      Download at: https://developer.apple.com/xcode/download/
      Or install Xcode via the App Store.
      Once installed, run:
        sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
        sudo xcodebuild -runFirstLaunch
    ✗ CocoaPods not installed.
        CocoaPods is used to retrieve the iOS and macOS platform side's plugin code that responds to your
        plugin usage on the Dart side.
        Without CocoaPods, plugins will not work on iOS or macOS.
        For more info, see https://flutter.dev/platform-plugins
      To install see https://guides.cocoapods.org/using/getting-started.html#installation for instructions.
[✓] Chrome - develop for the web
[✓] Android Studio (version 4.1)
[✓] VS Code (version 1.52.1)
[✓] Connected device (1 available)

```

#### 安装 Xcode（开发 ios App）

1.当我们升级完系统之后，可以直接在 app store 安装 xcode

这里，如果我们版本过低，会提示“不能将 Xcode 安装在 “Macintosh HD” 上，因为需要 macOS v10.15.4 或更高版本。”

2.安装完 xcode，重新`flutter doctor`，提示 cocoapods 未安装

这里我们使用 homebrew 进行安装

坑 1:如果你运行安装 homebrew 的命令之后，出现`curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused`

可以看这篇文章：https://blog.csdn.net/i_CodeBoy/article/details/107386756

```
brew install cocoapods
```

3.重新`flutter doctor`,提示还是有叉号

```
[!] Xcode - develop for iOS and macOS
    ✗ Xcode installation is incomplete; a full installation is necessary for iOS
      development.
      Download at: https://developer.apple.com/xcode/download/
      Or install Xcode via the App Store.
      Once installed, run:
        sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
        sudo xcodebuild -runFirstLaunch
```

根据提示,运行

```
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

```
sudo xcodebuild -runFirstLaunch
```

至此，全部完成。

```
wuhaomingdeMac-mini:homebrew kuro$ flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel master, 1.26.0-18.0.pre.90, on macOS 11.1 20C69 darwin-x64,
    locale zh-Hans-CN)
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 4.1)
[✓] VS Code (version 1.52.1)
[✓] Connected device (1 available)

• No issues found!
```

## 设置 IOS 模拟器

在命令行输入

```
open -a Simulator
```

打开之后，我们可以在菜单栏设置机型，比如 iphone12/iphone6 等，还可以通过右下角拖拽缩放模拟器大小。

## Android 开发环境设置

[developers](https://developer.android.com/)

### 安卓模拟器 在 AS 右上角可以添加设备

---

总结：[详情](https://www.devio.org/2019/04/03/development-environment-mac/)
