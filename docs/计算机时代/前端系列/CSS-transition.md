# CSS 之 transition

如果通过css实现一些属性变换实现动画的时候，这时候的动画可能比较生硬，效果不理想。
这时候我们就可以用采用`transition`属性来处理。

# 普通示例

## Demo

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS</title>
    <style>
        .my-element {
            width: 100px;
            height: 100px;
            margin-top: 100px;
            margin-left: 100px;
            background-color: red;
        }

        .my-element:hover {
            transform: rotate(45deg);
        }
    </style>
</head>

<body>
    <div class="my-element"></div>
</body>

</html>
```

## 对应效果

<video controls="controls"  src="/计算机时代/前端系列/assets/csstransition03.mov" width="100%"></video>

可以看出旋转效果很生硬。

# 使用transition示例

> transition: transform 1s ease-in-out;

## Demo

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS</title>
    <style>
        .my-element {
            width: 100px;
            height: 100px;
            margin-top: 100px;
            margin-left: 100px;
            background-color: red;
            transition: transform 1s ease-in-out;
        }

        .my-element:hover {
            transform: rotate(45deg);
        }
    </style>
</head>

<body>
    <div class="my-element"></div>
</body>

</html>
```

## 对应效果

<video controls="controls"  src="/计算机时代/前端系列/assets/csstransition04.mov" width="100%"></video>

使用`transition`指定`transform`属性变化需要`1s`，并且使用`ease-in-out`变化函数，
可以看出效果明显丝滑了很多。


更多请参考: https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function



done.