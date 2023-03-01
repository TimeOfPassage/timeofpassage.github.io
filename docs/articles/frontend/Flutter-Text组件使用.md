# Flutter-Text组件使用

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
this.style              : 设置文本样式
this.strutStyle         : 
this.textAlign          : 文本对齐方式
this.textDirection      : 文本方向
this.locale             : 
this.softWrap           : 是否自动换行，如果设置false，显示不全文本会按照overflow参数设置显示
this.overflow           : 显示不足时显示方式，如省略号 [clip：直接裁剪, fade：越来越透明, ellipsis：省略号结尾, visible：依然显示，此时将会溢出父组件]
this.textScaleFactor    : 文字缩放系数 1.5表示比原来的字大50%
this.maxLines           : 显示行数，是否换行显示
this.semanticsLabel     :
this.textWidthBasis     :
this.textHeightBehavior : 