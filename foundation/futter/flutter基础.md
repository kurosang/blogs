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
