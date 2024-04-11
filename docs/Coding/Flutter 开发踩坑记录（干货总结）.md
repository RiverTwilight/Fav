> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844904074622697480) ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/28/1708ae74a57ae575~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

Flutter 太好学了！BUG 真的太少了！ issues 只有 5000 多！也就那么亿点！简单得我都枯了！毕竟每次遇到问题，👴🏻 都是直接去找群里的法佬、低调、Alex 等几位大佬（🐶管理，此处小声哔哔）来解决，只要有大佬在，问题也就不大。虽然法佬经常说要学会看源码，但道理大家其实都懂，看源码也就图一乐，真正有 BUG 还是得找法佬。

不多哔哔，单写一篇文章，先记录它一手。本文记录 👴🏻 在 Flutter 开发中遇到的一些 BUG（as design），避免遗忘，如果正在看文章的你也遇到了，那咱们可以握个手。

容器宽高相关问题
--------

### Container 设置宽高不生效

一般是由于父级容器的 `constraints` 属性引起的，在 Flutter 中，子组件的大小会被父组件的 `constraints` 属性限制，例如

```
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: 100.0, // 最小宽度为 100 像素
    minHeight: 50.0 // 最小高度为 50 像素
  ),
  child: Container(
    height: 5.0,// 高度为 5 逻辑像素
    child: redBox 
  ),
)


```

上面的代码中，`Container` 组件设置高度为 5 像素，是无法生效的，因为父级容器已经设置了最小高度为 50 像素，所以 `Container` 组件的最终高度将会是 50 像素。

当然，这肯定不是我们想要的效果，我们就想让 `Container` 组件的最终高度是 5 像素怎么办？其实很简单，可以使用 `UnconstraindBox` 解除父级容器的 `constraints` 属性对子组件大小的限制。例如：

```
ConstrainedBox(
  constraints: BoxConstraints(
    minWidth: 100.0, // 最小宽度为 100 像素
    minHeight: 50.0 // 最小高度为 50 像素
  ),
  child: UnconstraintsBox(
    child: Container(
      height: 5.0, 
      child: redBox 
    ),
  ),
)


```

`UnconstrainedBox` 允许其子组件按照其自身的大小绘制，我们很少直接使用此组件，除非对于 Material 自带的一些组件，如 `Appbar` 的 icon 被官方限制了固定的大小，利用该组件可以解除限制，而一般情况下，我们在组件外面套一层布局类组件就可以解决需求，例如以下组件：

```
Row()
Column()
Align()
Center()
Flex()
Wrap()
Flow()
Stack()


```

### SignleChildScrollView 不满一屏高度时无法撑满全屏

其实和上面这个问题是相似的，可以使用布局类组件解决，或者用如下方式：

```
Container(
  alignment: Alignment.topLeft,
  child: SingleChildScrollView(),
),


```

如果你看过 `Container` 的源码你会发现其实设置 `alignment` 属性，和用 `Align` 组件是一回事，源码也是使用 `Align` 组件，这就是个语法糖，仅此而已。

说到语法糖，其实 `Center` 组件也是 `Align` 组件的语法糖，当你不给 `Align` 传递任何参数时，使用 `Center()` 和使用 `Align()` 是一模一样的效果，我的习惯是不管什么情况，都是只用`Align` 组件。

### 如何自定义 AppBar

上文提到过，Flutter 官方对 AppBar 的限制非常严格，连基本的高度都被写死了，这怎么能满足我们项目锦鲤所提出的花式需求呢？所以我在项目中除了使用了自带的 `SliverAppBar`，其他相关的 `AppBar` 组件基本没用。

自定义 `AppBar` 有两种方式：

第一种方式，使用 `Column` 及 `Expanded` 组件，提供我项目中的一个简单示例：

```
class VideoEditPage extends StatelessWidget {
  get appBar => DecoratedBox(
    decoration: BoxDecoration(
      color: currentTheme.primaryColor,
      boxShadow: tabBoxShadow,
    ),
    child: Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: <Widget>[
        Row(
          children: <Widget>[
            GestureDetector(
              behavior: HitTestBehavior.opaque,
              onTap: navigatorState.pop,
              child: Container(
                width: Screens.appBarHeight,
                height: Screens.appBarHeight,
                margin: EdgeInsets.only(right: setWidth(7)),
                child: Icon(
                  IcoMoon.arrowLeft,
                  size: setWidth(42),
                  color: currentTheme.primaryColorDark,
                ),
              ),
            ),
            Text('编辑视频', style: titleStyle),
          ],
        ),
        GestureDetector(
          behavior: HitTestBehavior.opaque,
          onTap: Routes.pushUploadCoverEditPage,
          child: Container(
            height: Screens.appBarHeight,
            alignment: Alignment.center,
            padding: EdgeInsets.symmetric(horizontal: setWidth(19)),
            child: Text('下一步', style: titleStyle),
          ),
        ),
      ],
    ),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Column(
          children: <Widget>[
            appBar,
            Expanded(child: listView)
          ],
        ),
      ),
    );
  }
}


```

第二种方式，使用 `Stack` 及 `Positioned` 组件，示例：

```
class MyApp extends StatelessWidget {
  get body => Stack(
    children: <Widget>[
      appBar,
      Positioned(
        left: 0,
        top: Screens.appBarHeight,
        right: 0,
        bottom: bottomAppBarHeight,
        child: listViewBox,
      ),
      bottomAppBar,
    ],
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: SizedBox.expand(child: body),
      ),
    );
  }
}


```

### Container 设置 borderRadius 不生效

设置 `borderRadius` 有两种做法，第一种使用 `Container` 等组件自带的 `borderRadius` 属性，第二种是，直接用 `ClipRRect` 等 clip 组件对容器进行裁剪，第二种比第一种更加暴力、消耗性能，但更有效。

例如给 `TabView` 的容器设置 `borderRadius`，你会发现无法生效，而使用 `ClipRRect` 则可以解决，我的理解是 `ClipRRect` 会直接裁剪成圆角形状，而 `BorderRadius` 的圆角外的弧形范围是透明的，类似 `css` 中的 `display:none` 与 `opaticy:0` 的区别，实际具体是什么原因，我也没有去细究，复制粘贴、能跑就行。

列表高度位置发生变化
----------

### TabBar 切换导致 PageView、ListView 等滚动组件位置改变

解决办法：给滚动组件加上 `key` 属性，用于保存位置信息，例如： `key: PageStorageKey(1)`

其实一般的 `ListView` 还无法满足我们日常开发中各种花式的需求，推荐使用法佬的 [NestedScrollView](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffluttercandies%2Fextended_nested_scroll_view "https://github.com/fluttercandies/extended_nested_scroll_view")

法佬已经给我们解决了很多奇怪的 BUG，还要什么自行车？

### push 页面返回后，导致页面重新 build 而引起位置变化

遇到这种情况很好解决，使用 `StatefulWidget` 混入 `AutomaticKeepAliveClientMixin` 保持页面状态，当路由 `pop` 后就不会引起重新 `build`，

例如：

```
class GalleryPage extends StatefulWidget {
  @override
  createState() => GalleryPageState();
}

class GalleryPageState extends State<GalleryPage> with AutomaticKeepAliveClientMixin {
  @override
  Widget build(BuildContext context) {
    super.build(context);
    return Scaffold(body: GridView.builder());
  }    

  @override
  bool get wantKeepAlive => true;
}


```

如上所示，如果在 `GalleryPage` 页 `push` 进一个新页面，再 `pop` 返回 `GalleryPage` 页时，状态会保持不变，列表不会重新 `build`。

元素显示层级问题
--------

可以认为 Flutter 中 `widget` 布局的层级关系是递进的，例如 `child` 的层级比父 `Widget` 层级更高， `Column`、`Row` 等组件的 `children` 中同级 `widget`，谁在后面谁的层级就更高，和 `Stack` 其 `children` 的层级关系相同。

### 显示隐藏的几种做法

第一种，利用 `IndexedStack` 组件控制层级，上面也提到过，子组件谁在后面谁的层级就高，Flutter 中虽然没有 `z-index` 这一说法，但其实原理和 css 的 `z-index` 是类似的，index 越大，层级越高，当然这里的 `IndexedStack` 的 `index` 属性是用来控制当前显示的某一个 children，只能显示一个。该方法常用于 APP 首页切换底部导航。

第二种，利用 `IgnorePointer` 及 `Opacity` 组件组合隐藏 widget，可以使用 `AnimationOpacity` 组件达到以前 `JQuery` 中常用的 `fadeIn` 效果。

第三种，利用 `Positioned` 或 `Transform.translate` 移动到屏幕外，需要显示时再移动回来，这种做法非常适合动画切换，例如视频进度条等效果。

第四种，利用 `Offstage` 组件，前三种都是利用视觉效果将元素隐藏起来，其实在布局上并未发生改变，而此组件就是类似于 css 中的 `display:none`，直接让元素在布局中隐藏，不会在布局上继续占用空间。

最后一种，在 build 方法中提前判断，不符合条件直接不渲染，或者返回空 box，这就类似于 HTML 中删除 dom 元素，我人没了，还显示个🔨，这是最恐怖的。

GestureDetector 设置 onTap 不生效
----------------------------

`Listener` 默认的 `behavior` 是 `HitTestBehavior.deferToChild`

如果 `Listener` 的子组件是一个 `Container`，这个 `Container` 不设置 `decoration` 的情况下，即透明背景色、无边框，则点击 `Container` 时，无法触发 `down、up` 等事件。

同理，`GestureDetector` 是对 `Listener` 的封装，无法触发 `onTap` 等事件也是必然的，那么解决办法也很简单，有以下两种解决办法：

```
1. 给 Container 设置 decoration
2. 将 behavior 属性设置为 opaque 或 translucent


```

调用 setState 或 markNeedsBuild 后报错
--------------------------------

### 第一种报错

> setState() or markNeedsBuild() called during build

遇到此提示，一般解决思路都是利用 `addPostFrameCallback` 来解决，例如：

```
WidgetsBinding.instance.addPostFrameCallback((_){
    _model.setOpacity(opacity);
});


```

### 第二种报错

> setState() called after dispose()

一般定时器在 app 返回桌面后仍在调用 `setState` 或 页面 pop 销毁后异步任务才完成，此时调用了 `setState` 必然会出现该提示，那么解决办法也很简单，判断生命周期再执行重构逻辑。

```
if (!mounted) return;
setState(() {
  // do somthing
});


```

### 动态更改 TabBar 的长度后 setState 报错

其实这个问题肯定是由于使用了 `SingleTickerProviderStateMixin` 造成的，解决方案有两种。

第一种是使用 `DefaultTabController` 来解决，这个方案比较适合大佬造轮子，因为需要自己写 `TabBar` 的切换效果，非常之麻烦。

第二种方案就是我目前正在使用的，非常简单，只需要将 `SingleTickerProviderStateMixin` 替换为 `TickerProviderStateMixin` 即可，相关代码如下：

```
class EntryPage extends StatefulWidget {
  @override
  createState() => _EntryPageState();
}

class _EntryPageState extends State<EntryPage> with TickerProviderStateMixin {

  TabController tabController;

  final tabs = <DanceSort>[
    DanceSort.fromJson({"id": -1, "name": "推荐"}),
    DanceSort.fromJson({"id": 0, "name": "关注"}),
  ];

  Future<void> getTabBar() async {
    final danceSorts = await EntryApi.getDanceSorts();
    if (danceSorts == null) return;
    tabs.addAll(danceSorts);
    tabController.dispose();
    tabController = TabController(length: tabs.length, vsync: this);
    setState(() {});
  }

  @override
  initState() {
    getTabBar();
    tabController = TabController(length: tabs.length, vsync: this);
    super.initState();
  }

  @override
  dispose() {
    tabController.dispose();
    super.dispose();
  }

  get tabBar => TabBar(
    controller: tabController,
    tabs: tabs.map<Tab>((v) => Tab(text: v.name)).toList(),
  );

  get tabBarView => TabBarView(
    controller: tabController,
    children: tabs.map<ListPage>((v) => ListPage(v.id)).toList(),
  );

  @override
  Widget build(BuildContext context) {
    return FloatingScrollView(
      tabBar: tabBar,
      tabBarView: tabBarView,
    );
  }
}


```

在 `initState` 时，先初始化本地默认的 tab，通过 Api 请求到服务端的 tab 数据后，再将原 `tabController` 销毁，生成一个新的 `tabController`，由于使用的是 `TickerProviderStateMixin`，所以并不会因为 `Single` 而报错。

为了便于理解，这个例子使用的 `setState` 来重新构建布局，其实完全可以使用 Provider 进行优化，我的项目也是全部使用 Provider 来进行管理的，利用 `Selector` 将构建范围缩小至最小，能很大地改善重构布局时的性能问题，例如上面 `tabBar` 部分可以换成：

```
get tabBar => Selector<HomeModel, TabController>(
  selector: (context, model) => model.tabController,
  builder: (context, controller, _) => TabBar(
    controller: controller,
    tabs: model.tabs.map<Tab>((v) => Tab(text: v.name)).toList(),
  ),
);


```

最后需要说明一点，当删除某个 `tab` 时，必须提前调整 `tabController` 的位置，例如以下代码是不生效的：

```
tabs.removeAt(3);
tabController.dispose();
tabController = TabController(length: tabs.length, vsync: this, initialIndex: 0);
setState(() {});


```

上面的代码，删除下标为 `3` 的 `tab` 后，重新赋值 `tabController`， 初始位置 `initialIndex` 的值为 `0`。

假设当前 `tab` 的 `index` 是 `3`，那么运行上面的代码后，你会发现无法生效，页面并未发生改变，依然停留在下标为 `3` 的列表。

所以正确的做法是提前调整位置，再重新赋值 `tabController`，例如：

```
tabs.removeAt(3);
tabController.index = 0;
tabController.dispose();
tabController = TabController(length: tabs.length, vsync: this);
setState(() {});


```

再补充最后一点，在上面的例子中， `tabBarView` 的生成方式如下：

```
get tabBarView => TabBarView(
  controller: tabController,
  children: tabs.map<ListPage>((v) => ListPage(v.id)).toList(),
);


```

`ListPage` 是一个 `StatefulWidget`，并混入了 `AutomaticKeepAliveClientMixin` 保持列表状态，这里可能会出现一些小问题。

例如动态删除某个 `tab` 后，重新赋值 `tabController`，从父级 `widget` 传递给 `ListPage` 的 `v.id` 依然是删除前的值，而不是新的 `id`，出现这个问题的原因是因为使用了 `initState` 或构造函数对 `State` 中的 `id` 进行了赋值，例如：

```
class ListPage extends StatefulWidget {
  final int id;
  ListPage(this.id);

  @override
  _ListPageState createState() => _ListPageState();
}

class _ListPageState extends State<ListPage> with AutomaticKeepAliveClientMixin {
  int id;

  @override
  void initState() {
    id = widget.id;
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);
    return ListView.builder(
      itemBuilder: (BuildContext context, int index) {
        return ListTile(
          title: Text(
            'state id $id, '
            'father widget id ${widget.id}, '
            'num $index'
          ),
        );
      }
    );
  }

  @override
  bool get wantKeepAlive => true;
}


```

运行上面代码后，删除某个 `tab` 再 `setState`，你会发现 `ListTile` 中显示的 `state id` 与 `father widget id` 竟然不一样，第一次遇到这个情况时我也惊了。

后来才发现 `initState` 或构造函数赋值只会在第一次 `build` 时生效，后面无论父级 `Widget` 重新 `setState` 多少次都不会对其产生影响。

知道了原因，解决办法也很简单，那就是在 `build` 中进行赋值，例如：

```
class _ListPageState extends State<ListPage> with AutomaticKeepAliveClientMixin {
  int id;
  
  @override
  Widget build(BuildContext context) {
    super.build(context);
    
    id = widget.id;
    return ListView.builder(
      itemBuilder: (BuildContext context, int index) {
        return ListTile(
          title: Text(
            'state id $id, '
            'father widget id ${widget.id}, '
            'num $index'
          ),
        );
      }
    );
  }

  @override
  bool get wantKeepAlive => true;
}


```

键盘相关问题
------

### 键盘弹出后将布局顶起来了，而不是遮住布局

解决办法：在 `scafold` 里设置 `resizeToAvoidBottomInset: false`，键盘会遮住布局，而不是顶起布局。

### 就想让键盘顶起布局，布局却溢出了怎么办？

溢出肯定是因为没有键盘时，整体高度没有一屏高，键盘出现了，却超出了一屏的高度。解决办法很简单，首先将布局使用 `SingleChildScrolleView` 之类的滚动组件包裹住，将布局改变为可滚动的，这样键盘弹出后布局就不会溢出了。

接着可以使用 `WidgetsBindingObserver` 类来监听键盘弹起事件，每次弹起键盘后会自动触发 `didChangeMetrics` 钩子，在该钩子里执行逻辑即可，例如将 `SingleChildScrolleView` 的当前位置调整至最底部，相关代码如下：

```
import 'package:flutter/material.dart';

class Demo extends StatefulWidget {
  @override
  createState() => _DemoState();
}

class _DemoState extends State<Demo> with WidgetsBindingObserver {

  final _scrollController = ScrollController();
  final _phoneController = TextEditingController();

  FocusNode _phoneFocusNode = FocusNode();
  FocusScopeNode _focusScopeNode;

  get _phoneTextFiled => TextField(
    controller: _phoneController,
    focusNode: _phoneFocusNode,
    keyboardType: TextInputType.phone,
    maxLength: 11,
    decoration: InputDecoration(
      hintText: '请输入手机号',
      border: InputBorder.none,
      counterText: '',
    ),
  );

  void handlePostFrame() {
    if (!_phoneFocusNode.hasFocus) {
      print('requestFocus');
      _focusScopeNode.requestFocus(_phoneFocusNode);
    }
    print('jumpTo');
    _scrollController.jumpTo(_scrollController.position.maxScrollExtent);
  }

  @override
  void initState() {
    WidgetsBinding.instance.addObserver(this);
    super.initState();
  }

  @override
  void didChangeMetrics() {
    WidgetsBinding.instance.addPostFrameCallback(handlePostFrame);
    super.didChangeMetrics();
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
}


```

### 键盘弹起和收回会引起页面重新 build

我的项目中有一个接近 1 万行代码的视频详情页，全部使用 `Provider` 进行状态管理，如果键盘弹起、收回引起重新 build，就可能出现一些奇怪的 BUG，比如当前的滚动组件在屏幕中的位置发生变化。

我的解决方案是利用 `showBottomSheet` 方法，页面中展示的 `TextField` 上盖一层透明遮罩，使用户无法点击，而点击遮罩时，则触发 `showBottomSheet`， push 进一个新的路由，弹起键盘，却不会引起重新 build，收起键盘时，则会 pop 回页面，其实视觉上一直都保持在同一页面中，和普通的弹起键盘没区别，并且性能也非常棒，相关代码如下：

```
  get textField => TextField(
    autofocus: true,
    cursorColor: currentTheme.hoverColor,
    cursorWidth: 1.0,
    textInputAction: TextInputAction.done,
    style: TextStyle(
      color: currentTheme.primaryColorLight,
      fontSize: setSp(32),
    ),
    decoration: InputDecoration(
      hintText: '发一句友善的评论来见证当下吧',
      hintStyle: TextStyle(fontSize: setSp(28)),
      contentPadding: EdgeInsets.symmetric(horizontal: setWidth(31)),
      filled: true,
      fillColor: currentTheme.primaryColorDark,
      border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(setWidth(30)),
          borderSide: BorderSide.none
      ),
    ),
    onSubmitted: (value) {},
  );

  Widget buildTextFieldPage(BuildContext context) {
    return SizedBox.expand(
      child: Stack(
        alignment: Alignment.bottomLeft,
        children: <Widget>[
          Positioned.fill(
            child: GestureDetector(
              behavior: HitTestBehavior.opaque,
              onTap: () => Navigator.pop(context),
              child: Container(color: Colors.black.withOpacity(.5)),
            ),
          ),
          buildInput(),
        ],
      ),
    );
  }

  buildInput({hasTextField = true}) {
    Widget child;

    child = hasTextField
        ? Container(
            decoration: BoxDecoration(
              color: currentTheme.backgroundColor,
              borderRadius: BorderRadius.circular(setWidth(31)),
            ),
            child: textField,
          )
        : GestureDetector(
            onTap: () {
              showBottomSheet(
                context: context,
                backgroundColor: Colors.transparent,
                builder: buildTextFieldPage,
              );
            },
            child: Container(
              decoration: BoxDecoration(
                color: currentTheme.backgroundColor,
                borderRadius: BorderRadius.circular(setWidth(31)),
              ),
            ),
          );

    return Container(
      height: setWidth(103),
      padding: EdgeInsets.symmetric(
        vertical: setWidth(20),
        horizontal: setWidth(25),
      ),
      decoration: BoxDecoration(
        border: Border(top: commentDivider),
        color: currentTheme.primaryColor,
      ),
      child: Row(
        children: <Widget>[
          Expanded(child: child),
          Container(
            width: setWidth(66),
            padding: EdgeInsets.only(left: setWidth(25)),
            alignment: Alignment.center,
            child: Icon(
              IcoMoon.send,
              color: currentTheme.hoverColor.withOpacity(.5),
              size: setWidth(42),
            ),
          ),
        ],
      ),
    );
  }


```

相关效果如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/28/1708afc1f79c4da1~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

上图使用了 `showBottomSheet` 弹出了键盘，其实也可以使用 `showDialog` 完成相同功能，将 `GestureDetector` 的 `onTap` 回调替换为如下：

```
showDialog(
  context: context,
  builder: buildTextFieldPage,
);


```

需要注意的是，`showDialog` 的 `builder` 回调需要使用 `Material` 或 `Scaffold` 包住页面，否则样式会发生异常：

```
Widget buildTextFieldPage(BuildContext context) {
  return Scaffold(
    backgroundColor: Colors.transparent,
    body: SizedBox.expand(
      child: Stack(
        alignment: Alignment.bottomLeft,
        children: <Widget>[
          Positioned.fill(
            child: GestureDetector(
              behavior: HitTestBehavior.opaque,
              onTap: () => Navigator.pop(context),
              child: Container(color: Colors.black.withOpacity(.5)),
            ),
          ),
          buildInput(autoFocus: true),
        ],
      ),
    ),
  );
}


```

相关实现效果如下图：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/171718021e0cb101~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

### TextField 如何根据内容自适应高度

`TextField` 在最大行数为一行时，可以直接通过父级 `Container` 限制 `TextField` 的高度，不需要设置额外属性，但是在多行高度时，这种做法就不太灵验。

可以看到上图中评论框会根据输出的内容自动调整高度，并且限制了最大行数为 4，而这种需求又是特别常见的，我是如何做到的呢？

首先，需要设置 `TextField` 的 `maxLine` 属性为 `null`，这样 `TextField` 就会根据内容自动调整高度

接着，设置 `decoration: InputDecoration(isDense:true)`，这样 `TextField` 将允许我们限制 `TexField` 的最大高度

最后，利用父级容器限制 `TextField` 的最大高度，例如

```
Container(
  constraints: BoxConstraints(maxHeight: 200),
  child: TextField(),
)


```

通过以上设置后， `TextField` 的高度就是根据内容自动适应，并且会被限制最大高度，但是还需要说明一点，如何保证最大高度刚好是 4 行的高度？

由于 `TextField` 没有属性是用来限制每行高度的，所以这就需要我们自己计算 `contentPadding` 了，例如：

```
decoration: InputDecoration(
  isDense: true,
  contentPadding: EdgeInsets.symmetric(vertical: 20),
}


```

### TextField 设置 border 不生效

TextField 的 `border` 有如下 3 种，需要针对性地设置，只设置一个是无法生效的：

```
decoration: InputDecoration(border enabledBorder focusBorder)


```

ps：设置 `maxLength` 属性后，`decoration` 里需要设置 `counterText: ''`，否则默认会附带一个统计字数的样式。

路由跳转相关问题
--------

### push、pop 常见需求

例如浏览记录中有如下 4 个页面，当前页面为 `d`：

```
a->b->c->d


```

在当前页面使用 `Navigator.popUtil(context, ModalRoute.withName('a'))`，可以直接返回至 `a` 页面，并销毁 `b`、`c` 页面。

在当前页面使用 `Navigator.pushNamedAndRemoveUntil(context, 'e', (route) => false)`，可以进入 `e` 页面之前，销毁所有历史记录，即 `e` 页面变成第一页，`e` 页面里无法继续 `pop` 返回上一页。

### 如何在 initState 阶段就能使用 context

Flutter 有一些需要使用 `context` 的方法，例如 `Theme.of(context)`、`Navigator.of(context)` 等等，一般情况下，我们需要在 build 方法中才能拿到 `context`，如果我们在定义类的属性时就直接使用 `Theme.of(context)` 显然是没办法做到的，编辑器都会直接提示语法错误。

解决办法也很简单，首先创建一个文件 `constants.dart` 用于保存项目的全局变量，在 `Contants` 类中声明一个 `navigatorKey`：

```
class Constants {
  static GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();
}


```

接着在 `MaterailApp` 下挂载 `navigatorKey` ：

```
MaterialApp(navigatorKey: Constants.navigatorKey)


```

最后在 `constants.dart` 中声明如下 3 个 `getter`：

```
NavigatorState get navigatorState => Constants.navigatorKey.currentState;
BuildContext get currentContext => navigatorState.context;
ThemeData get currentTheme => Theme.of(currentContext);


```

由于所有组件都是挂载在 `MaterialApp` 下的，所以我们在任何一个组件中引入了 `constants.dart`，就可以使用这 3 个 `getter`，用法如下：

```
class LoginPage extends StatelessWidget {
  final box = GestureDetector(
    onTap: navigatorState.pop,
    child: Text('返回上一页'),
  );
  final userModel = Provider.of<UserModel>(currentContext, listen: false);
  final primaryColor = currentTheme.primaryColor;
}


```

上面的代码等价于：

```
class LoginPage extends StatelessWidget {
  Widget box;
  UserModel userModel;
  Color primaryColor;
  
  build(context) {
    box = GestureDetector(
      onTap: Navigator.of(context).pop,
      child: Text('返回上一页'),
    );
    userModel = Provider.of<UserModel>(context, listen: false);
    primaryColor = Theme.of(context).primaryColor;
  }
}


```

网络请求相关
------

### 如何封装 Dio

网络请求大家基本都是使用 `Dio`，我在项目上手后遇到的第一个问题就是如何对 `Dio` 进行封装，下面提供一下我的做法（仅供参考）：

（1）`lib` 目录下创建一个 `api` 目录，用于存放网络请求相关的文件。

（2）`api` 目录下创建一个 `api_path.dart` 用于存放项目接口的地址配置信息，内容大致如下：

```
const baseUrl = 'http://app.lcgod.com/';
const version = '1.0';

class ApiPath {
  static const messages = 'messages';
  static const loginWithSMS = 'login/sms';
  static const loginWithPassword = 'login/password';
  static const loginWithWeChat = 'login/wechat';
  static const loginWithQQ = 'login/qq';
  static const loginWithWeibo = 'login/weibo';
  static const videos = 'videos';
  static const users = 'users';

  static followUser(int id) => '$users/$id/follow_users';
  static videoCollect(int id) => '$videos/$id/collect_users';
  static videoLike(int id) => '$videos/$id/like_users';
  static videoShare(int id) => '$videos/$id/share_users';
  static videoComments(int id) => '$videos/$id/comments';
}


```

（3）`api` 目录下创建一个 `api.dart` 作为 `Dio` 的基类，内容大致如下：

```
import 'dart:async';
import 'dart:io';
import 'dart:convert';

import 'package:dio/dio.dart';
import 'package:http/io_client.dart';
import 'package:http_client_helper/http_client_helper.dart';

import 'api_path.dart';

export 'login_api.dart';
export 'user_api.dart';

class Api {
  static Dio _dio;

  static void init() {
    HttpClient http = HttpClient();
    http.badCertificateCallback = (cert, host, port) => true;
    IOClient client = IOClient(http);
    HttpClientHelper().set(client);
    _dio = Dio()
      ..options.baseUrl = baseUrl
      ..options.headers['accept'] = 'application/vnd.vhiphop.v$version+json'
      ..interceptors.add(InterceptorsWrapper(onRequest: _handleRequest));
  }

  static RequestOptions _handleRequest(RequestOptions options) {
    String fullPath = options.baseUrl + options.path;

    if (options.method == 'GET' && options.queryParameters.length > 0) {
      List params = [];
      options.queryParameters.forEach((k, v) => params.add('$k=$v'));
      fullPath += '?' + params.join('&').toString();
      print(fullPath);
    }

    final time = DateTime.now().millisecondsSinceEpoch ~/ 1000;
    options.headers.addAll({
      'x-date': '$time',
      'x-token': '123456',
      'x-user-id': Constants.user.id == null ? '' : '${Constants.user.id}',
      'x-user-token': Constants.user.token ?? '',
    });

    return options;
  }

  static final _fetchTypes = <String, Function>{
    'post': _dio.post,
    'put': _dio.put,
    'patch': _dio.patch,
    'delete': _dio.delete,
    'head': _dio.head,
  };

  static Future<dynamic> head(url, {Map<String, dynamic> data}) async {
    return await _fetch('head', url, data);
  }

  static Future<dynamic> get(url, {Map<String, dynamic> data}) async {
    return await _fetch('get', url, data);
  }

  static Future<dynamic> post(url, {Map<String, dynamic> data}) async {
    return await _fetch('post', url, data);
  }

  static Future<dynamic> put(url, {Map<String, dynamic> data}) async {
    return await _fetch('put', url, data);
  }

  static Future<dynamic> patch(url, {Map<String, dynamic> data}) async {
    return await _fetch('patch', url, data);
  }

  static Future<dynamic> delete(url, {Map<String, dynamic> data}) async {
    return await _fetch('delete', url, data);
  }

  static Future<dynamic> _fetch(method, url, data) async {
    try {
      final Response response = method == 'get'
        ? await _dio.get(url, queryParameters: data)
        : await _fetchTypes[method](url, data: data);
      return response.data;
    } catch (e) {
      final error = (e is DioError
          && e.response != null
          && e.response.statusCode == 403)
        ? e.response.data
        : {"message": "服务器网络繁忙，请稍后再试", "status_code": 1001};

      showTip(error['message']);

      throw error;
    }
  }
}


```

（4）`api` 目录下创建具体请求逻辑的文件，例如 `login_api.dart`，用于登录逻辑的接口逻辑放在该文件中：

```
import 'package:vhiphop/constants/constants.dart';

import 'api.dart';
import 'api_path.dart';

class LoginApi {
  static Future<dynamic> loginWithPassword({phone, password}) async {
    print('开始手机密码登录');
    var isError = false;
    final user = await Api.post(ApiPath.loginWithPassword, data: {
      'phone': phone,
      'password': password,
    }).catchError((_) => isError = true);
    return isError ? true : User.fromJson(user);
  }

  static Future<dynamic> loginWithWeChat(code) async {
    print('开始微信授权登录');
    var isError = false;
    final user = await Api.post(ApiPath.loginWithWeChat, data: {
      'code': code,
    }).catchError((_) => isError = true);
    return isError ? true : User.fromJson(user);
  }

  static Future<dynamic> checkCodeAndGetToken({
    String phone,
    String code
  }) async {
    print('开始校验验证码并获取token');
    var isError = false;
    final user = await Api.get(ApiPath.messages, data: {
      'phone': phone,
      'code': code,
    }).catchError((_) => isError = true);
    return isError ? true : User.fromJson(user);
  }
}


```

（5）调用示例：

```
Future<void> _handleEnterButtonTap() async {
  if (_isLogging) return;
  if (_code < 1 || _code > 999999) return showTip('验证码错误');

  _isLogging = true;
  showLoading('正在登录');
  final user = await LoginApi.loginWithSMS(
    phone: _phone,
    code: _code,
    nation: _nation,
  );
  _isLogging = false;
  if (user == true) return;
  dismissAllToast();

  Constants.user = user;
  Constants.saveUser();
  Routes.pop();
}


```

补充：

*   所有接口请求类都会由 `api.dart` 导出，再由 `constants.dart` 导出，那么项目中所有文件只需要引入一个 `constants.dart` 就可以拿到各个请求类。
*   所有网络请求遇到错误都会统一进行处理，例如我的代码中是统一使用 `OkToast` 弹出提示框，提示具体信息。

### Dio 请求的 `content-type` 问题

使用 Dio 进行 HTTP 请求时，请求头 `content-type` 的默认值是

```
application/json; charset=utf-8


```

如果返回头的 `content-type` 是

```
application/json


```

Dio 将自动解析返回 json 数据为 Dart 相应的数据类型，而不需要手动地调用 `jsonDecode` 方法，所以客户端、服务端的统一使用 `application/json` 作为 `content-type`，他好我也好。

### Android 打包后无法进行网络请求

在我第一次使用 Flutter 打包项目时遇到了这个问题，最后发现是没有网络请求的权限，类似的，储存读取本地文件时可能也会有类似问题，这种问题设置权限就可以解决了。

在 `android/app/src/profile/AndroidManifest.xml` 中

以及 `android/app/src/main/AndroidManifest.xml` 两个文件的 `manifest` 标签内添加如下子标签即可：

```
<uses-permission android: />
<uses-permission android: />
<uses-permission android: />
<uses-permission android: />


```

如果需要动态申请权限，可以使用 `permission_handler` 插件。

Mac 环境 build 时的错误
-----------------

提示如下：

Automatically assigning platform `iOS` with version `9.0` on target `Runner` because no platform was specified. Please specify a platform for this target in your Podfile.

解决办法是：删除 `pod` 文件中 `platform`前的 `#`

因为没有做过原生开发，所以对于这种 build 问题真的是一脸茫然，最开始遇到过几次类似错误，我通过网上搜索答案、群里问大佬来解决，非常之麻烦。所以后来我在 Mac 环境 build 产生错误时，都是直接重建项目，把逻辑代码复制进新项目里，再重新 build 就不会发生各种乱七八糟看不懂的错误了，效率也快。

如何监听 App 返回桌面事件
---------------

我需要当 app 返回桌面时暂停视频的播放，从桌面返回 app 后再继续播放，解决方案如下：

```
class _DemoState extends State<Demo> with WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    print('app lifecycle state: $state');
    if (state == AppLifecycleState.inactive) {
      _playerModel.pausePlayer();
    } else if (state == AppLifecycleState.resumed) {
      if (_homeModel.isFindPage) _playerModel.startPlayer();
    }
    super.didChangeAppLifecycleState(state);
  }
}



```

WidgetsBindingObserver 这个类我经常使用，例如监听键盘弹起事件也会用到这个类。

对于类中的属性和方法的定义规范的一些建议
--------------------

*   不引用其他属性的成员，定义为属性
    
*   引用其他属性，且不接收参数的成员，定义为 getter
    
*   引用其他属性，且接受参数的成员，定义为 function
    

全屏相关设置
------

强制竖屏：

```
void initState() {
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown
  ]);
  super.initState();
}


```

强制横屏：

```
initState() {
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight
  ]);
  super.initState();
}


```

Transform 3D 转换
---------------

推荐使用 `Transform` 组件来完成动画效果，例如 `Transform.translate` 和 `Transform.scale` 可以完成位置、缩放的变化， `Transform.rotate` 可以完成旋转角度的变化。

`Transform.rotate` 和 `RotateBox` 都可以完成旋转功能，他们之间有什么区别？

使用 `RotateBox` 渲染 widget 是在 layout 阶段，渲染完毕后就会占用实际位置，而 `Transform` 组件则是在 layout 之后的绘制阶段， `Transform` 只是一个视觉效果，实际所占空间大小是 transform 变化之前所占用的空间大小，所以重新渲染 `Transform.rotate` 组件比重新渲染 `RotateBox` 开销更小。

Flutter 的 `Transform` 组件的这个特性和 CSS 的 `transform` 属性非常相似，都可以用来提升动画性能。

不过做视频全屏功能时，可以用 `IndexedStack` + `RotateBox` 替代 push 一个横屏的路由的做法，`RotateBox` 它会使容器填充全屏，而 `IndexedStack` 可以控制是否显示全屏，这里如果使用 `Transform` 则无法填充全屏，因为容器的宽高在 layout 时就已经确定了，所以只能使用 `RotateBox`。

### 视频镜像翻转

我在项目中不仅使用 RotatedBox 完成视频全屏功能，还利用了 `Transform` 来完成镜像翻转功能，写法如下：

```
Selector<VideoModel, bool>(
  selector: (context, model) => model.isMirror,
  builder: (context, isMirror, child) => Transform(
    alignment: Alignment.center,
    transform: Matrix4.identity()..setEntry(3, 2, 0.006)..rotateY(isMirror ? math.pi : 0),
    child: child,
  ),
  child: FijkView(
    player: model.player,
    color: Colors.black,
    panelBuilder: (player, context, size, pos) => emptyBox,
  ),
)


```

原理很简单，FijkView 是 fijkplayer 提供的视频容器，我将视频容器以中心位置为圆心，沿 Y 轴做一个 180 度的旋转，即可满足需求。

`setEntry` 用于设置透视，否则将无法看到 Y 轴及 X 轴的立体转换效果

`rotateY` 则与 css 中的 `rotateY` 是相同含义，即沿 Y 轴旋转。在 css 中可以设置 `transform: rotateY(180deg)` 来达到相同的效果。

状态栏相关设置
-------

隐藏状态栏：

```
import 'package:flutter/services.dart';

void toggleFullscreen() {
  _isFullscreen = !_isFullscreen;
  _isFullscreen
      ? SystemChrome.setEnabledSystemUIOverlays([])
      : SystemChrome.setEnabledSystemUIOverlays(SystemUiOverlay.values);
}


```

改变状态栏颜色，则需要使用插件：`flutter_statusbarcolor`，下面是用法示例：

```
// 改变状态栏背景颜色，默认改变为透明
Future<void> changeStatusColor({Color color: Colors.transparent}) async {
  try {
    await FlutterStatusbarcolor.setStatusBarColor(
      color,
      animate: true,
    );
    FlutterStatusbarcolor.setStatusBarWhiteForeground(true);
    FlutterStatusbarcolor.setNavigationBarWhiteForeground(true);
  } on PlatformException catch (e) {
    debugPrint(e.toString());
  }
}


```

下面介绍一个用法，我的 home 页使用 `indexStack` 组件包含了 4 个 tab 页，每次更改 tab 会改变 `currentHomeTab` 的值，但不会触发重新 build，而由于路由 `push` 或 `pop` 又会触发重新 build，所以如果需要当进入 home 页的 `发现 tab 页` 时改变为黑色状态栏，则可以用下面这种做法：

```
// 在发现页的 build 方法里进行判断
@override
Widget build(BuildContext context) {
  if (ModalRoute.of(context).isCurrent && currentHomeTab == '发现') {
    changeStatusColor(color: Colors.black);
  }
}


```

fijkplayer 秒开、进度跳转等优化
---------------------

fijkplayer 默认情况下，进度跳转、播放可能会有性能问题，针对这些问题，可以进行以下优化：

```
_player.setDataSource(_video.src);
await _player.applyOptions(
    FijkOption()
      ..setFormatOption('flush_packets', 1)
      ..setFormatOption('analyzeduration', 1)
      ..setCodecOption('skip_loop_filter', 48)
      ..setPlayerOption('start-on-prepared', 1)
      ..setPlayerOption('packet-buffering', 0)
      ..setPlayerOption('framedrop', 1)
      ..setPlayerOption('enable-accurate-seek', 1)
      ..setPlayerOption('find_stream_info', 0)
      ..setPlayerOption('render-wait-start', 1)
);
await _player.prepareAsync();


```

参考链接：

[IjkPlayer 起播速度优化](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fc8c755ed61ee "https://www.jianshu.com/p/c8c755ed61ee")

[IjkPlayer 播放器秒开优化以及常用 Option 设置](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F843c86a9e9ad "https://www.jianshu.com/p/843c86a9e9ad")

LayoutBuilder 相关的实践
-------------------

### 如何实现微信朋友圈、哔哩哔哩评论的多行文本收起、展开功能

我写了下面这个工具类，简单、好用得我都枯了，原理是利用先 `LayoutBuilder` 判断是否超出指定的行数，如果超出则返回 `Column`，如果未超出则返回原 widget

```
import 'package:flutter/material.dart';

class ExpandableText extends StatefulWidget {
  final String text;
  final int maxLines;
  final TextStyle style;
  final bool expand;
  final TextStyle markerStyle;
  final String atName;

  const ExpandableText(this.text, {
    Key key,
    this.maxLines,
    this.style,
    this.markerStyle,
    this.expand = false,
    this.atName = '',
  }) : super(key: key);

  @override
  createState() => _ExpandableTextState();

}

class _ExpandableTextState extends State<ExpandableText> {

  bool expand;
  TextStyle style;
  int maxLines;

  @override
  void initState() {
    expand = widget.expand;
    style = widget.style;
    maxLines = widget.maxLines;
    super.initState();
  }

  Widget buildOrdinaryText() {
    final text = widget.text;
    return LayoutBuilder(builder: (_, size) {
      final tp = TextPainter(
        text: TextSpan(text: text, style: style),
        maxLines: maxLines,
        textDirection: TextDirection.ltr,
      );
      tp.layout(maxWidth: size.maxWidth);

      if (!tp.didExceedMaxLines) return Text(text, style: style);

      return Builder(
        builder: (context) => Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Text(text, maxLines: expand ? null : widget.maxLines, style: style),
            GestureDetector(
              onTap: () {
                expand = !expand;
                (context as Element).markNeedsBuild();
              },
              child: Text(
                expand ? '收起' : '展开',
                style: widget.markerStyle,
              ),
            ),
          ],
        ),
      );
    });
  }

  Widget buildAtText() {
    return LayoutBuilder(builder: (_, size) {
      final tp = TextPainter(
        text: TextSpan(text: '回复 @${widget.text}：', style: style),
        maxLines: maxLines,
        textDirection: TextDirection.ltr,
      );
      tp.layout(maxWidth: size.maxWidth);

      if (!tp.didExceedMaxLines) return Text.rich(
        TextSpan(
          children: [
            TextSpan(text: '回复 '),
            TextSpan(text: '@${widget.atName}', style: widget.markerStyle),
            TextSpan(text: '：${widget.text}'),
          ],
        ),
        style: style,
      );

      return Builder(
        builder: (context) => Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Text.rich(
              TextSpan(
                children: [
                  TextSpan(text: '回复 '),
                  TextSpan(text: '@${widget.atName}', style: widget.markerStyle),
                  TextSpan(text: '：${widget.text}'),
                ],
              ),
              maxLines: expand ? null : widget.maxLines,
              style: style,
            ),
            GestureDetector(
              onTap: () {
                expand = !expand;
                (context as Element).markNeedsBuild();
              },
              child: Text(
                expand ? '收起' : '展开',
                style: widget.markerStyle,
              ),
            ),
          ],
        ),
      );
    });
  }

  @override
  build(context) => widget.atName == '' ? buildOrdinaryText() : buildAtText();
}


```

调用方法如下：

```
Container(
  padding: EdgeInsets.only(top: setWidth(6), bottom: setWidth(11)),
  alignment: Alignment.centerLeft,
  child: ExpandableText(
    reply.content,
    maxLines: 4,
    style: commentTextStyle,
    markerStyle: commentMarkerStyle,
    atName: reply.isDirect > 0 ? '' : reply.pNickname,
  ),
),


```

相关效果如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/28/1708ae74a4a2b29b~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

### 监听父级 widget 的实际宽高信息

LayoutBuilder 的作用非常大，可以用它来监听某个 widget 的宽高信息，我在项目中遇到了 一个需求，需要根据某个 widget 的高度来弹出 BottomSheet，而这个 widget 的高度是可以滑动改变的，那么 `LayoutBuilder` 就派上用场了，做法如下：

需要监听的 `widget` 是 `Body()` 组件，给 Body() 组件套上一个 `Stack`

```
get body => Stack(
  children: <Widget>[
    Body(),
    BodyLayout(model),
  ],
);


```

然后用 `BodyLayout` 组件来监听：

```
import 'package:flutter/material.dart';

import 'package:vhiphop/provider/video/video_model.dart';

class BodyLayout extends StatelessWidget {

  final VideoModel model;
  BodyLayout(this.model);

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (_, BoxConstraints constraints) {
      model.bottomSheetDy = constraints.maxHeight;
      return emptyBox;
    });
  }
}


```

当 `Body()` 组件高度发生变化时，会触发 `LayoutBuilder` 的 `builder` 回调函数，在此函数中将高度信息传递给 `model` ，那么每次弹出 `BottomSheet` 之前，我就可以从 model 中拿到高度，以设置 BottomSheet 的高度。

底部弹出动画的两种实现方式
-------------

这种动画在 App 中是很常见的效果，例如 App 分享功能，点击分享按钮后，会从页面底部弹出分享组件。

第一种，利用 `showModalBottomSheet`，相关实现代码如下：

```
  void showShareBottomSheet() {
    showModalBottomSheet(
      elevation: 0,
      backgroundColor: currentTheme.highlightColor,
      context: context,
      builder: (context) => Container(
        width: Screens.width,
        decoration: BoxDecoration(color: currentTheme.primaryColor),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            Container(
              alignment: Alignment.bottomLeft,
              height: setWidth(59),
              padding: EdgeInsets.only(left: setWidth(42)),
              child: Text(
                '分享',
                style: TextStyle(
                  fontSize: setSp(32),
                  color: currentTheme.highlightColor,
                ),
              ),
            ),
            Container(
              height: setWidth(206),
              padding: EdgeInsets.only(top: setWidth(33), left: setWidth(33)),
              alignment: Alignment.topLeft,
              decoration: BoxDecoration(
                border: Border(
                  bottom: BorderSide(
                    width: setWidth(.7),
                    color: currentTheme.dividerColor,
                  ),
                ),
              ),
              child: Row(
                children: <Widget>[
                  shareIconOfQQ,
                  shareIconOfQQZone,
                  shareIconOfWeChat,
                  shareIconOfWeChatMoments,
                  shareIconOfMicroBlog,
                ],
              ),
            ),
            Container(
              height: setWidth(206),
              padding: EdgeInsets.only(top: setWidth(33), left: setWidth(33)),
              alignment: Alignment.topLeft,
              child: Row(
                children: <Widget>[
                  shareIconOfLink,
                ],
              ),
            ),
            GestureDetector(
              onTap: () {
                Navigator.pop(context);
              },
              child: Container(
                width: Screens.width,
                height: setWidth(125),
                alignment: Alignment.center,
                decoration: BoxDecoration(
                  border: Border(
                    top: BorderSide(
                      width: setWidth(10),
                      color: currentTheme.backgroundColor,
                    ),
                  ),
                ),
                child: Text(
                  '取消',
                  style: TextStyle(
                    fontSize: setSp(36),
                    color: currentTheme.highlightColor,
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }


```

### 使用 translate 实现

我在项目中使用 `showModalBottomSheet` 时发现动画有点卡顿，可能是测试手机不行，只花了 1000 大洋，但咱是个倔强穷人，非要找一种性能更好的方式，那就是 `translate` 了。

这种方法比 `showModalBottomSheet` 动画性能更高，在我 1000 大洋的测试机 debug 模式下都非常地丝滑流畅，只是代码实现更复杂一点，并且需要依赖 Provider 来更新，我比较喜欢这种方式。

整个页面都使用 `Stack` 构建，而 bottomSheet 与遮罩 box 则使用 `Positioned` 定位至页面底部：

```
get body => Stack(
  children: <Widget>[
    page,
    Positioned(
      left: 0,
      bottom: 0,
      right: 0,
      child: bottomSheetBox,
    ),
    Positioned(
      left: 0,
      top: 0,
      right: 0,
      bottom: shareBottomSheetHeight,
      child: bottomSheetBoxMask,
    ),
  ],
);


```

接着使用我定义的一个工具类，名字叫 `AnimatedTranslateBox`，我发现 Animated 家族有各种动画组件，比如 `AnimatedPadding`、`AnimatedPositioned` 等等，唯独没有 `Translate`，不知道官方是什么意思，可能他们觉得 `Positioned` 来调整位置就够用了叭，可是 `translate` 动画性能更高，它不香吗？没关系，咱自己造了一个，代码如下：

```
import 'package:flutter/material.dart';

class AnimatedTranslateBox extends StatefulWidget {
  AnimatedTranslateBox({
    Key key,
    this.dx,
    this.dy,
    this.child,
    this.curve = Curves.linear,
    this.duration = const Duration(milliseconds: 200),
    this.reverseDuration,
  });

  final double dx;
  final double dy;
  final Widget child;
  final Duration duration;
  final Curve curve;
  final Duration reverseDuration;

  @override
  createState() => _AnimatedTranslateBoxState();
}

class _AnimatedTranslateBoxState extends State<AnimatedTranslateBox>
    with SingleTickerProviderStateMixin {

  AnimationController controller;
  Animation<double> animation;
  Tween<double> tween;

  void _updateCurve() {
    animation = widget.curve == null
      ? controller
      : CurvedAnimation(parent: controller, curve: widget.curve);
  }

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      duration: widget.duration,
      reverseDuration: widget.reverseDuration,
      vsync: this,
    );
    tween = Tween<double>(begin: widget.dx ?? widget.dy);
    _updateCurve();
  }

  @override
  void didUpdateWidget(AnimatedTranslateBox oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.curve != oldWidget.curve) _updateCurve();
    controller
      ..duration = widget.duration
      ..reverseDuration = widget.reverseDuration;
    if ((widget.dx ?? widget.dy) != (tween.end ?? tween.begin)) {
      tween
        ..begin = tween.evaluate(animation)
        ..end = widget.dx ?? widget.dy;
      controller
        ..value = 0.0
        ..forward();
    }
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }

  @override
  build(context) => AnimatedBuilder(
    animation: animation,
    builder: (context, child) => widget.dx == null
        ? Transform.translate(
            offset: Offset(0, tween.animate(animation).value),
            child: child,
          )
        : Transform.translate(
            offset: Offset(tween.animate(animation).value, 0),
            child: child,
          ),
    child: widget.child,
  );
}


```

调用很简单，使用 `Selector` 依赖 model 中的布尔值，用于控制显示隐藏：

```
get bottomSheetBox => Selector<VideoModel, bool>(
  selector: (context, model) => model.showBottomSheet,
  builder: (context, show, child) => AnimatedOpacity(
    opacity: show ? 1 : 0,
    curve: show ? Curves.easeOut : Curves.easeIn,
    duration: bottomSheetDuration,
    child: AnimatedTranslateBox(
      dy: show ? 0 : bottomSheetHeight,
      curve: show ? Curves.easeOut : Curves.easeIn,
      duration: bottomSheetDuration,
      child: child,
    ),
  ),
  child: Container(
    height: bottomSheetHeight,
    child: bottomSheet,
  ),
);


```

每当 `dx` 或 `dy` 的值发生改变，`AnimatedTranslateBox` 的 child 就会根据 `dx` 或 `dy` 的值进行 y 轴 或 x 轴的移动动画。

相关的效果如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/28/1708b07cd338e0ca~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

Provider 调用问题
-------------

我发现如果在 `MaterialApp` 下全局挂载了 Provider ，则在 Home 页初始化完成前，是无法使 Provider 的，例如：

```
class MyApp extends StatelessWidget {

  final _userModel = UserModel();
  final _homeModel = HomeModel();

  Widget build(BuildContext context) {
    return OKToast(
      dismissOtherOnShow: true,
      child: MultiProvider(
        providers: [
          ChangeNotifierProvider.value(value: _userModel),
          ChangeNotifierProvider.value(value: _homeModel),
        ],
        child: Selector<ThemeModel, ThemeData>(
          selector: (context, model) => model.theme,
          builder: (context, theme, child) => MaterialApp(
            navigatorKey: Constants.navigatorKey,
            debugShowCheckedModeBanner: false,
            theme: theme,
            initialRoute: '/',
            routes: {
              '/': (context) => HomePage(),
            },
          ),
        ),
      ),
    );
  }
}


```

上面的代码声明了 `MultiProvider`，如果在首页做如下调用：

```
@override
initState() {
  _model = Provider.of<HomeModel>(context);
  _userModel = Provider.of<UserModel>(context);
  super.initState();
}


```

则会报错：

```
I/flutter ( 8380): ══╡ EXCEPTION CAUGHT BY WIDGETS LIBRARY ╞═══════════════════════════════════════════════════════════
I/flutter ( 8380): The following assertion was thrown building Builder:
I/flutter ( 8380): dependOnInheritedWidgetOfExactType<_DefaultInheritedProviderScope<HomeModel>>() or
I/flutter ( 8380): dependOnInheritedElement() was called before _HomePageState.initState() completed.
I/flutter ( 8380): When an inherited widget changes, for example if the value of Theme.of() changes, its dependent
I/flutter ( 8380): widgets are rebuilt. If the dependent widget's reference to the inherited widget is in a constructor
I/flutter ( 8380): or an initState() method, then the rebuilt dependent widget will not reflect the changes in the
I/flutter ( 8380): inherited widget.
I/flutter ( 8380): Typically references to inherited widgets should occur in widget build() methods. Alternatively,
I/flutter ( 8380): initialization based on inherited widgets can be placed in the didChangeDependencies method, which
I/flutter ( 8380): is called after initState and whenever the dependencies change thereafter.


```

提示 `initState` 必须调用完成，才能使用 `Provider.of` 来获取祖先节点的 model，非要使用怎么办？办法也很简单， `of` 方法有一个属性值 `listen`，默认值为 `true`，将此值设置为 `false` 则不会建立与 Provider 的依赖关系，其实我在 Provider 的手册中也发现，建议在 `initState` 方法中调用 `of` 时，将 `listen` 设置为 `false`：

```
@override
initState() {
  _userModel = Provider.of<UserModel>(context, listen: false);
  _model = Provider.of<HomeModel>(context, listen: false);
  super.initState();
}


```

如何实现网易云音乐、QQ 音乐播放页面的背景图片模糊效果
----------------------------

分析一下，其实这种效果特别简单，首先放大背景图片，其次对图片进行高斯模糊，直接上代码：

```
import 'package:flutter/material.dart';
import 'dart:ui';

main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  final image = Image.asset(
    'assets/images/test.jpg',
    fit: BoxFit.cover,
    width: 200,
    height: 200,
  );
  
  get blurImage => ClipRect(
    child: Stack(
      children: <Widget>[
        Transform.scale(
          scale: 1.5,
          child: image,
        ),
        BackdropFilter(
          filter: ImageFilter.blur(sigmaX: 5, sigmaY: 5),
          child: Container(
            width: 200,
            height: 200,
            alignment: Alignment.center,
            color: Colors.black.withOpacity(.3),
            child: Text(
              '1 个内容',
              style: TextStyle(
                fontSize: 24,
                color: Colors.white,
              ),
            ),
          ),
        ),
      ],
    ),
);
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo app',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: Scaffold(
        appBar: AppBar(title: Text('blur image demo')),
        body: Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Container(
                  margin: EdgeInsets.only(bottom: 30),
                  child: image,
                ),
                blurImage,
              ],
            ),
          ],
        )
      ),
    );
  }
}


```

这个效果其实没什么难度，主要的知识点在于 `BackdropFilter` 组件默认的模糊效果是全屏的，必须使用 `ClipRect` 进行裁剪，而且 `Transform` 的几个命名构造函数，如 `Transform.translate` 带来的效果是在绘制阶段发生的，会超出 `widget` 实际占用的空间，也需要使用 `ClipRect` 进行裁剪，最后的效果图如下：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/28/1708ae749e3286e5~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)