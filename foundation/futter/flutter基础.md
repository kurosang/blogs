## flutter 基础

### 创建一个项目

flutter create xxx

### 如何使用 flutter 包和插件？

https://pub.dev/

在这个网上搜想要的插件

### StatelessWidget（无状态组件）

常用衍生出来的组件：Container、Text、Icon、CloseButton、BackButton、Chip、Divider、Card、AlertDialog

Chip: 材料设计中一个非常有趣的小部件，具体看 https://material.io/components/chips

```
/// StatelessWidget与基础组件
class LessGroupPage  extends StatelessWidget {
  TextStyle textStyle = TextStyle(fontSize: 20);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '如何使用flutter plugin',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text('StatelessWidget与基础组件'),),
        body: Container(
          decoration: BoxDecoration(color: Colors.white),
          alignment: Alignment.center,
          child: Column(
            children: <Widget>[
              Text('I am Text',style: textStyle,),
              Icon(Icons.phone_iphone,size: 50,color:Colors.red),
              CloseButton(),
              BackButton(),
              Chip(
                avatar: Icon(Icons.people),
                  label: Text('asdasd')
              ),
              Divider(
                height:100,//容器高度，不是线的高度，可以理解为上下撑开的高度，但是本身分割线高度没有设置
                indent: 10,//左侧间距
                color: Colors.orange,
              ),
              Card(
                //带有圆角，阴影，边框等效果的卡片
                color:Colors.blue,
                elevation: 5, //阴影
                margin: EdgeInsets.all(10),//上下左右都是10的边距
                child: Container(
                  padding: EdgeInsets.all(10),
                  child: Text('i am card',style: textStyle,),
                ),
              ),
              AlertDialog(
                title: Text('没有闪'),
                content: Text('大衣了'),
              )
            ],
          ),
        ),
      ),
    );
  }
}

```

### StatefulWidget（有状态组件）

常用：

- MaterialApp：材料设计 app 的组件，通常放在根结点
- Scaffold：flutter 封装的，带有 appBar，底部导航栏，侧边栏的组件
- AppBar：顶部导航栏
- BottomNavigationBar：底部导航栏
- RefreshIndicator：刷新指示器，下拉刷新要配合 ListView 使用
- Image：图片组件
- TextField：输入框组件
- PageView

```
/// StatefulWidget与基础组件
class StateFulGroup extends StatefulWidget {
  @override
  _StateFulGroupState createState() => _StateFulGroupState();
}

class _StateFulGroupState extends State<StateFulGroup> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    TextStyle textStyle = TextStyle(fontSize: 20);
    return MaterialApp(
      title: 'StatefulWidget与基础组件',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text('StatefulWidget与基础组件'),),
        bottomNavigationBar: BottomNavigationBar(currentIndex: _currentIndex,
          onTap: (i){
          setState(() {
            _currentIndex = i;
          });
        },items: [
          BottomNavigationBarItem(
              icon: Icon(Icons.home,color:Colors.grey),
              activeIcon: Icon(Icons.home,color: Colors.blue,),
              label: '首页'
          ),
          BottomNavigationBarItem(
              icon: Icon(Icons.list,color:Colors.grey),
              activeIcon: Icon(Icons.list,color: Colors.blue,),
              label: '列表'
          )
        ],),
        floatingActionButton: FloatingActionButton(
          onPressed: null,
          child:Text('点我')
        ),
        body: _currentIndex == 0? RefreshIndicator(child: ListView(
          children: <Widget>[
            Container(
              decoration: BoxDecoration(color: Colors.white),
              alignment: Alignment.center,
              child: Column(
                children: <Widget>[
                  Image.network('https://resource.ghzs.com/image/article/large/601e48996aaf2d6fec12349d.png',
                  width: 100,
                  height: 100,),
                  TextField(
                    //输入文本的样式
                    decoration: InputDecoration(
                      contentPadding: EdgeInsets.fromLTRB(5, 0, 0, 0),
                      hintText: '请输入',
                      hintStyle: TextStyle(fontSize: 15)
                    ),

                  ),
                  Container(
                    height: 100,
                    margin: EdgeInsets.only(top:10),
                    decoration: BoxDecoration(color: Colors.lightBlueAccent),//container背景
                    child:PageView(
                      children: [
                        _item('Page1',Colors.deepPurple),
                        _item('Page2',Colors.blue),
                        _item('Page3',Colors.green),
                      ],
                    )
                  )
                ],
              ),
            )
          ],
        ), onRefresh: _handleRefresh):Text("list"),
      ),
    );
  }

  // 异步方法
  Future<Null> _handleRefresh() async{
    await Future.delayed(Duration(milliseconds: 200));
    return null;
  }
  // 方法生成组件
  _item(String title, Color color) {
    return Container(
      alignment: Alignment.center,
      decoration: BoxDecoration(color:color),
      child: Text(title,style: TextStyle(fontSize: 22,color:Colors.white)),
    );
  }
}
```

### flutter 布局开发

相关组件：

**Container**

**RenderObjectWidget**

- SingleChildRenderObjectWidget（单节点布局组件）

  - Opacity：改变透明度的组件
  - ClipOval：将布局裁剪成圆形
  - clipRRect：将布局裁剪成方形
  - PhysicalModel：用来将布局显示成不同形状
  - Align：其中，center 控制居中
  - Padding
  - SizedBox 约束布局大小
  - FractionallySizedBox 约束水平方向或垂直方向的伸展

- MultiChildRenderObjectWidget（多节点布局组件）

  - Stack：所有的布局都一个叠一个
  - Flex：column 从上到下的。Row 从左到右。排列布局
  - Wrap：与 Row 类似，但可以换行
  - Flow：少用

**ParentDataWidget**

- Positioned：固定 view 的位置，通常和 stack 搭配使用

- Flexible
  - Expanded：展开

```
/// Flutter布局开发
class FlutterLayoutPage extends StatefulWidget {
  @override
  _FlutterLayoutPageState createState() => _FlutterLayoutPageState();
}

class _FlutterLayoutPageState extends State<FlutterLayoutPage> {
  int _currentIndex = 0;

  @override
  Widget build(BuildContext context) {
    TextStyle textStyle = TextStyle(fontSize: 20);
    return MaterialApp(
      title: 'Flutter布局开发',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text('Flutter布局开发'),),
        bottomNavigationBar: BottomNavigationBar(currentIndex: _currentIndex,
          onTap: (i){
            setState(() {
              _currentIndex = i;
            });
          },items: [
            BottomNavigationBarItem(
                icon: Icon(Icons.home,color:Colors.grey),
                activeIcon: Icon(Icons.home,color: Colors.blue,),
                label: '首页'
            ),
            BottomNavigationBarItem(
                icon: Icon(Icons.list,color:Colors.grey),
                activeIcon: Icon(Icons.list,color: Colors.blue,),
                label: '列表'
            )
          ],),
        floatingActionButton: FloatingActionButton(
            onPressed: null,
            child:Text('点我')
        ),
        body: _currentIndex == 0? RefreshIndicator(child: ListView(
          children: <Widget>[
            Container(
              decoration: BoxDecoration(color: Colors.white),
              alignment: Alignment.center,
              child: Column(
                children: <Widget>[
                  Row(
                    children: [
                      ClipOval(
                        child: SizedBox(
                          width: 100,
                          height: 100,
                          child: Image.network('https://resource.ghzs.com/image/article/large/601e48996aaf2d6fec12349d.png'),
                        ),
                      ),
                      Padding(padding: EdgeInsets.all(10),
                      child: ClipRRect(
                        // 圆角
                        borderRadius: BorderRadius.all(Radius.circular(10)),
                        child: Opacity(
                          opacity: 0.6,//60%透明度
                          child: Image.network('https://resource.ghzs.com/image/article/large/601e48996aaf2d6fec12349d.png',
                            width: 100,
                            height: 100,
                          ),
                        ),
                      ),)
                    ],
                  ),
                  Container(
                      height: 100,
                      margin: EdgeInsets.all(10),
                      child:
                      PhysicalModel(
                        color: Colors.transparent,
                        borderRadius: BorderRadius.circular(20),
                        clipBehavior: Clip.antiAlias,// 裁切的边是否抗锯齿
                        child: PageView(
                          children: [
                            _item('Page1',Colors.deepPurple),
                            _item('Page2',Colors.blue),
                            _item('Page3',Colors.green),
                          ],
                        ),
                      )
                  ),
                  Column(
                    children: [
                      FractionallySizedBox(
                        widthFactor: 1,
                        child: Container(
                          decoration: BoxDecoration(color:Colors.green),
                          child: Text("宽度撑满"),
                        ),
                      )
                    ],
                  )
                ],
              ),
            ),
            Stack(
              children: [
                Image.network('https://resource.ghzs.com/image/article/large/601e48996aaf2d6fec12349d.png',
                  width: 100,
                  height: 100,
                ),
                Positioned(
                  left: 0,
                  bottom: 0,
                  child: Image.network('https://resource.ghzs.com/image/article/large/601e48996aaf2d6fec12349d.png',
                  width: 36,
                  height: 36,
                ),)
              ],
            ),
            Wrap(
              // 创建wrap布局，从左向右进行排列，会自动换行
              spacing: 8, // 水平边距
              runSpacing: 6,// 垂直边距
              children: [
                _chip('Flutter'),
                _chip('进阶'),
                _chip('布局'),
              ],
            )
          ],
        ), onRefresh: _handleRefresh):Column(
          children: [
            Text("list"),
            Expanded(child: Container(
              decoration:BoxDecoration(color:Colors.red) ,
              child: Text('拉伸填满高度'),
            ))
          ],
        ),
      ),
    );
  }

  // 异步方法
  Future<Null> _handleRefresh() async{
    await Future.delayed(Duration(milliseconds: 200));
    return null;
  }
  // 方法生成组件
  _item(String title, Color color) {
    return Container(
      alignment: Alignment.center,
      decoration: BoxDecoration(color:color),
      child: Text(title,style: TextStyle(fontSize: 22,color:Colors.white)),
    );
  }

  _chip(String label) {
    return Chip(label: Text(label),
    avatar: CircleAvatar(
      backgroundColor: Colors.blue.shade900,
      child: Text(label.substring(0,1),style: TextStyle(fontSize: 10)),
    ),);
  }
}


```

### 如何创建和使用 flutter 的路由和导航

两种方式跳转

有配置 routes 则使用第一种

```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(title: Text("如何创建和使用flutter的路由和导航"),),
        body: RouteNavigator(),
      ),
      routes: <String, WidgetBuilder>{
        'plugin': (BuildContext context) => PluginUse(),
        'less': (BuildContext context) => LessGroupPage(),
        'ful': (BuildContext context) => StateFulGroup(),
        'layout': (BuildContext context) => FlutterLayoutPage(),
      },
    );
  }
}

class RouteNavigator extends StatefulWidget {
  @override
  _RouteNavigatorState createState() => _RouteNavigatorState();
}

class _RouteNavigatorState extends State<RouteNavigator> {
  bool byName = false;
  @override
  Widget build(BuildContext context) {
    return Container(
      child: Column(children: [
        SwitchListTile(title: Text('${byName?'':'不'}通过路由名跳转'),value: byName, onChanged: (val){
          setState(() {
            byName = val;
          });
        }),
        _item('如何使用flutter plugin', PluginUse(), 'plugin'),
        _item('StatelessWidget与基础组件', LessGroupPage(), 'less'),
        _item('StatefulWidget与基础组件', StateFulGroup(), 'ful'),
        _item('Flutter布局开发', FlutterLayoutPage(), 'layout'),
      ],),
    );
  }

  _item(String title, page, String routeName) {
    return Container(
      child: ElevatedButton(onPressed: (){
        // 两种方式跳转
        // 有配置routes则使用第一种
        if(byName){
          Navigator.pushNamed((context), routeName);
        }else{
          Navigator.push(context, MaterialPageRoute(builder: (context)=> page));
        }
      },
      child: Text(title),)
    );
  }
}


```

返回上一页

```
 appBar: AppBar(title: Text('StatefulWidget与基础组件'),leading: GestureDetector(
          onTap:(){
            Navigator.pop(context);
          },
          child: Icon(Icons.arrow_back),
        )),
```

GestureDetector 好像是手势

### 如何检测用户手势以及处理点击事件

两个例子：

- 输出各种手势事件以及输出顺序
- 拖动小球

```
import 'package:flutter/material.dart';

// 如何检测用户手势以及处理点击事件
class GesturePage extends StatefulWidget {
  @override
  _GesturePageState createState() => _GesturePageState();
}

class _GesturePageState extends State<GesturePage> {

  String printString = "";

  double moveX = 0, moveY = 0;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        primarySwatch: Colors.blue
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text("如何检测用户手势以及处理点击事件?"),
          leading: GestureDetector(
            onTap: (){
              Navigator.pop(context);
            },
            child: Icon(Icons.arrow_back),
          ),
        ),
        body: FractionallySizedBox(
          widthFactor: 1,
          child: Stack(
            children: [
              Column(
                children: [
                  GestureDetector(
                    onTap: () => _printMsg('点击'),
                    onDoubleTap: () => _printMsg('双击'),
                    onLongPress: () => _printMsg('长按'),
                    onTapCancel: () => _printMsg('取消'),
                    onTapUp: (e) => _printMsg('松开'),
                    onTapDown: (e) => _printMsg('按下'),
                    child: Container(
                      padding: EdgeInsets.all(60),
                      decoration: BoxDecoration(color: Colors.blueAccent),
                      child: Text('点我',style: TextStyle(fontSize: 36,color: Colors.white),),
                    ),
                  ),
                  Text(printString)
                ],
              ),
              Positioned(
                //跟着手指滑动的小球
                left: moveX,
                top: moveY,
                child: GestureDetector(
                  // 拖动回调
                  onPanUpdate: (e) => _doMove(e),
                  child: Container(
                    width: 72,
                    height: 72,
                    decoration: BoxDecoration(color:Colors.amber,borderRadius: BorderRadius.circular(36)),
                  ),
                ),
              )
            ],
          ),
        ),
      )
    );
  }

  _printMsg(String s){
  setState(() {
    printString += ' ,$s';
  });
  }

  _doMove(DragUpdateDetails e) {
    setState(() {
      moveY += e.delta.dy;
      moveX += e.delta.dx;
    });
  }
}

```

### 如何导入和使用 Flutter 的资源文件

图片、字体文件

所以在`pubspec.yaml`文件中，声明图片路径

```
  assets:
     - images/avatar.png
```

然后使用方法：

```
 Image(
    width:100,
    height:100,
    image: AssetImage('images/avatar.png')
)
```

### 如何打开第三方应用？

插件：url_launcher

https://pub.flutter-io.cn/packages/url_launcher

安装完之后，使用

```
import 'package:url_launcher/url_launcher.dart';

_launchURL() async {
  const url = 'http://www.baidu.com';
  if(await canLaunch(url)){
    await launch(url);
  }else{
    throw 'Could not launch $url';
  }
}

_openMap()async {
  // Android
  const url = 'geo:52.32,4.917'; // APP提供者提供的schema
  if(await canLaunch(url)){
    await launch(url);
  }else{
    // ios
    const url = 'http://maps.apple.com/?ll=52.32,4.917';
    if(await canLaunch(url)){
      await launch(url);
    }else{
      throw 'Could not launch $url';
    }
  }
}
```

上面的方法：一个是打开浏览器 demo，一个是打开地图 demo

### Flutter 页面生命周期实战指南

主要讲一下 StatefulWidget 的声明周期，因为 StatelessWidget（无状态组件）只有 createElement 和 build 两个周期

StatefulWidget 的生命周期按照时期不同可以分为三组：

- 初始化时期（createState、initState）
- 更新时期（didChangeDependencies、build、didUpdateWidget）
- 销毁期（deactivate、dispose）

拓展：

https://www.devio.org/io/flutter_app/img/blog/flutter-widget-lifecycle.png

https://flutterbyexample.com/lesson/stateful-widget-lifecycle

```
import 'package:flutter/material.dart';

/// Flutter页面生命周期
class WidgetLifecycle extends StatefulWidget {
  ///createState
  ///当我们构建一个新的StatefulWidget时，这个会立即调用
  ///并且这个方法必须被覆盖
  @override
  _WidgetLifecycleState createState() => _WidgetLifecycleState();
}

class _WidgetLifecycleState extends State<WidgetLifecycle> {

  int _count = 0;

  ///这是创建Widget时调用的除构造方法外的第一个方法：
  ///类似于Android的：onCreate() 与 IOS的 viewDidLoad()
  ///在这个方法中通常会做一些初始化工作，比如channel的初始化、监听器的初始化等
  @override
  void initState() {
    print('----initState----');
    super.initState();
  }

  ///当依赖的class _WidgetLifecycleState extends State<WidgetLifecycle>中的State对象改变时调用：
  ///a.在第一次构建widget时，在initState() 之后立即调用此方法;
  ///b.如果StatefulWidgets依赖于InheritedWidget，那么当当前State所依赖InheritedWidget中的变量改变时会再次调用它
  ///拓展：InheritedWidget可以高效的将数据在Widget树中向下传递、共享。
  ///参考：https://book.flutterchina.club/chapter7/inherited_widget.html
  @override
  void didChangeDependencies() {
    print('----didChangeDependencies----');
    super.didChangeDependencies();
  }

  ///这是一个必须实现的方法，在这里实现你要呈现的页面内容：
  ///它会在在didChangeDependencies()之后立即调用；
  ///另外当调用setState后也会再次调用该方法；
  @override
  Widget build(BuildContext context) {
    print('----build----');
    return Scaffold(
      appBar: AppBar(title: Text("Flutter页面生命周期"),
      leading: BackButton(),),
      body: Center(
        child: Column(
          children: [
            ElevatedButton(onPressed: (){
              setState(() {
                _count+=1;
              });
            },child: Text('点我',style: TextStyle(fontSize: 26),),),
            Text(_count.toString())
          ],
        ),
      ),
    );
  }

  /// command + n 然后选择override methods
  /// 这是一个不常用的生命周期方法，当父组件需要重新绘制时才会调用；
  /// 该方法会携带一个oldWidget参数，可以将其与当前的widget进行对比以便执行一些额外的逻辑，如：
  /// if(oldWidget.xxx != widget.xxx)...
  @override
  void didUpdateWidget(WidgetLifecycle oldWidget) {
    print('----didUpdateWidget----');
    super.didUpdateWidget(oldWidget);
  }

  ///很少使用，在组件被移除时调用在dispose之前调用
  @override
  void deactivate() {
    print('----deactivate----');
    super.deactivate();
  }

  ///常用，组件被销毁时调用；
  ///通常在该方法中执行一些资源的释放工作比如，监听器的卸载，channel的销毁等
  @override
  void dispose() {
    print('----dispose----');
    super.dispose();
  }
}

```

最常用：initState build dispose

### 如何获取 Flutter 应用的生命周期

1.需要在 State 对象后面带上 WidgetsBindingObserver

2.initState 初始化添加

3.记得 depose 页面销毁时移除

```
import 'package:flutter/material.dart';

/// 如何获取 Flutter 应用的生命周期
/// WidgetsBindingObserver：是一个Widgets绑定观察器，通过它我们可以监听应用的生命周期、语言等的变化
class AppLifecycle extends StatefulWidget {
  @override
  _AppLifecycleState createState() => _AppLifecycleState();
}

class _AppLifecycleState extends State<AppLifecycle> with WidgetsBindingObserver{
  /// 在初始化的时候，将这个类添加到监听器里面
  @override
  void initState() {
    WidgetsBinding.instance.addObserver(this);
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Flutter 应用的生命周期'),leading: BackButton(),),
      body: Container(
        child: Text(' Flutter 应用的生命周期'),
      ),
    );
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);
    print('state = $state');
    if(state == AppLifecycleState.paused){
      print('App进入后台');
    }else if(state == AppLifecycleState.resumed){
      print('App进入前台');
    }else if(state == AppLifecycleState.inactive){
      //不常用：当应用程序处于非活动状态，并且未接收用户输入时调用，比如：来了个电话
    }else if(state == AppLifecycleState.detached){
      // AppLifecycleState.suspended已更改为AppLifecycleState.detached
      //不常用：应用程序被挂起时调用，它不会在ios上触发
    }
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
}

```

### 如何修改 Flutter 应用的主题

如果我们需要动态的修改主题，需要我们把 MyApp 的 stateless 改为 stateful 组件

然后设置 MaterialApp 里的 theme 的 brightness 来设置暗夜模式

```
@override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        brightness: Brightness.dark, // 夜间模式
        primarySwatch: Colors.blue,
      ),
      .....
```

### 如何自定义字体？

https://fonts.google.com/

1.首先下载需要的字体，然后解压，把 ttf 文件放到项目`根目录/fonts/`里面

2.在`pubspec.yaml`上加上

```
  fonts:
    - family: AkayaTelivigala
      fonts:
        - asset: fonts/AkayaTelivigala-Regular.ttf
```

记得加完之后，要运行`flutter pub get`,然后重启 app

3.使用

- 全局使用

```
 theme: ThemeData(
    fontFamily: 'AkayaTelivigala',
    ...
```

- 局部使用

```
Text('切换模式change',style: TextStyle(fontFamily: 'AkayaTelivigala'),)
```

### 拍照 APP 开发

https://pub.flutter-io.cn/packages/image_picker

因为 image_picker 这个插件用到安卓 x 的兼容库

兼容 androidx：https://flutter.cn/docs/development/packages-and-plugins/androidx-compatibility

ios 配置：

ios/Runner/Info.plist

```
...
<string>$(FLUTTER_BUILD_NUMBER)</string>
<key>LSRequiresIPhoneOS</key>
<true/>
...
<key>NSCameraUsageDescription</key>
<string>在这里配置相机使用说明</string>
<key>NSMicrophoneUsageDescription</key>
<string>在这里配置录音使用说明</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>在这里配置相册的使用说明</string>
...
<key>UILaunchStoryboardName</key>
<string>LaunchScreen</string>
....
```

可以参考这里：https://github.com/flutter/plugins/blob/master/packages/image_picker/image_picker/example/ios/Runner/Info.plist

### flutter 项目学习

https://github.com/flutter/flutter/tree/master/examples

https://github.com/flutter/samples

https://github.com/nisrulz/flutter-examples

https://github.com/iampawan/FlutterExampleApps

### 图片开发核心技能

https://www.devio.org/2019/04/11/flutter-image-widget/

**Image 支持如下几种类型的构造函数**

- new Image - 用于从 ImageProvider 获取图像；
- new Image.asset - 使用 key 从 AssetBundle 获得的图像；
- new Image.network - 从网络 URL 中获取图片；
- new Image.file - 从本地文件中获取图片；
- new Image.memory - 用于从 Uint8List 获取图像；

> 在加载项目中的图片资源时，为了让 Image 能够根据像素密度自动适配不同分辨率的图片，请使用 AssetImage 指定图像，并确保在 widget 树中的“Image” widget 上方存在 MaterialApp，WidgetsApp 或 MediaQuery 窗口 widget。

Image 支持以下类型的图片：JPEG, PNG, GIF, Animated GIF, WebP, Animated WebP, BMP, 和 WBMP。

**如何加载网络图片？**

要加载网络图片，我们需要使用 Image.network 构造方法：

```
Image.network(
  'https://www.devio.org/img/avatar.png',
)
```

**如何加载静态图片，以及处理不同分辨率的图片**

- 在 pubspec.yaml 文件中声明图片资源的路径；

```
assets:
 - images/my_icon.png
```

- 使用 AssetImage 访问图片；

```
Image(
  height: 26,
  width: 26,
  image: AssetImage(my_icon.png),
),
```

除了我们使用 Image 的构造方法手动指定 AssetImage 之外，还可通过 Image.asset 来加载静态图片：

```
Image.asset(my_icon.png,
	width: 26,
	height: 26,
)
```

**如何加载本地图片**

1.加载完整路径的本地图片

```
import 'dart:io';
Image.file(File('/sdcard/Download/Stack.png')),
```

2.加载相对路径的本地图片

- 第一步：在 pubspec.yaml 中添加 path_provider 插件；

https://pub.dev/packages/path_provider

- 第二步：导入头文件

```
import 'dart:io';
import 'package:path_provider/path_provider.dart';

//Image.file(File('/sdcard/Download/Stack.png')),
FutureBuilder(future: _getLocalFile("Download/Stack.png"),
  builder:  (BuildContext context, AsyncSnapshot<File> snapshot) {
    return snapshot.data != null ? Image.file(snapshot.data) : Container();
  })
)
//获取SDCard的路径：
 Future<File> _getLocalFile(String filename) async {
    String dir = (await getExternalStorageDirectory()).path;
    File f = new File('$dir/$filename');
    return f;
  }
```

**如何设置 Placeholder**
