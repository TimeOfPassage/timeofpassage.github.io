# #一些Flutter组件解析

# 核心组件MaterialApp&Scaffold

```dart
// 应用脚手架
const MaterialApp({
  Key key,
  this.navigatorKey, // 可以定义全局key，避免子组建需要context信息
  this.home, // 主界面的内容 widget
  this.routes = const <String, WidgetBuilder>{}, // 带 router 和路由跳转有关
  this.initialRoute, // 初始化路由，当配置onGenerateRoute，则不需要配置此项
  this.onGenerateRoute, // 路由生成
  this.onUnknownRoute, // 404路由
  this.navigatorObservers = const <NavigatorObserver>[], 
  this.builder,
  this.title = '', // *类似标题
  this.onGenerateTitle, // 主要用于多语言情况下，需要根据当前语言替换 title，需要使用该值
  this.color, // 主题色，如果该值未设置，取 theme.primaryColor,未设置 theme 则取蓝色
  this.theme, // App 的主题风格，包括主题色，按钮默认颜色等等
  this.locale, // 带 locale 的和多语言适配相关
  this.localizationsDelegates,
  this.localeListResolutionCallback,
  this.localeResolutionCallback,
  this.supportedLocales = const <Locale>[Locale('en', 'US')], // 支持语言设置
  this.debugShowMaterialGrid = false, 
  this.showPerformanceOverlay = false, // 显示性能浮层
  this.checkerboardRasterCacheImages = false,
  this.checkerboardOffscreenLayers = false,
  this.showSemanticsDebugger = false,
  this.debugShowCheckedModeBanner = true, // debug 模式下，是否显示 DEBUG 标示横幅
})

// Scaffold 页面脚手架
const Scaffold({
  Key key,
  this.appBar, // 界面顶部的那条栏，这边需要返回一个 AppBar 实例
  this.body, // 界面的内容部分
  this.floatingActionButton, // 悬浮部分，可以通过 floatingActionButtonLocation 设置位置
  this.floatingActionButtonLocation,
  this.floatingActionButtonAnimator,
  this.persistentFooterButtons,
  this.drawer, // 侧滑抽屉部分，从左侧滑出(应该和语言有关，和文字方向同向)
  this.endDrawer, // 侧滑抽屉部分，从右侧滑出
  this.bottomNavigationBar, // 底部导航栏，就是通常看到的底部 TAB 切换部件
  this.bottomSheet, // 展示从底部弹出的，起到提示作用的，通过 showModalBottomSheet 展示
  this.backgroundColor, // 界面的背景色
  this.resizeToAvoidBottomPadding = true, // 避免 body 被底部弹出部件填充，例如输入法键盘
  this.primary = true, // 当前的 Scaffold 是否需要被展示在屏幕最上层
})
```

# Text组件使用

```dart
const Text(
    this.data, {
    Key key,
    this.style,
    this.strutStyle,
    this.textAlign,
    this.textDirection,
    this.locale,
    this.softWrap,
    this.overflow,
    this.textScaleFactor,
    this.maxLines,
    this.semanticsLabel,
    this.textWidthBasis,
    this.textHeightBehavior,
  }) : assert(
         data != null,
         'A non-null String must be provided to a Text widget.',
       ),
       textSpan = null,
       _applyTextScaleFactorToWidgetSpan = true,
       super(key: key);
```

**必填参数** 

	this.data: 文本内容

**选填参数** 

	this.style : 设置文本样式 

	this.textAlign : 文本对齐方式 

	this.textDirection : 文本方向 

	this.locale : 国际化

	this.softWrap : 是否自动换行，如果设置false，显示不全文本会按照overflow参数设置显示 

	this.overflow : 显示不足时显示方式，如省略号 [clip：直接裁剪, fade：越来越透明, ellipsis：省略号结尾, visible：依然显示，此时将会溢出父组件] 

	this.textScaleFactor : 文字缩放系数 1.5表示比原来的字大50% 

	this.maxLines : 显示行数，是否换行显示
