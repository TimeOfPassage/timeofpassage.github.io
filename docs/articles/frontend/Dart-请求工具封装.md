# Dart-请求工具封装

工具通过对dio进行二次封装

## 首先pubspec.yaml添加dio依赖

```
dependencies:
  dio: ^4.0.6
```

执行`dart pub get`下载依赖。


## 新建utils目录及request_util.dart文件

具体内容如下

```dart
import 'package:dio/dio.dart';

class RequestUtil {
  Dio? _dio;
  static RequestUtil? _instance;

  RequestUtil._internal([BaseOptions? options]) {
    if (options != null) {
      _dio = Dio(options);
    } else {
      _dio = Dio(
        BaseOptions(
          baseUrl: "https://www.tianqiapi.com",
          connectTimeout: 6000,
          receiveTimeout: 6000,
          sendTimeout: 6000,
        ),
      );
    }
    // // 请求队列，可针对请求做统一处理
    // _dio?.interceptors.add(InterceptorsWrapper(
    //   onRequest: (options, handler) {
    //     // print('REQUEST[${options.method}] => PATH: ${options.path}');
    //     return handler.next(options);
    //   },
    // ));
    // // 日志拦截器
    // _dio?.interceptors.add(LogInterceptor());
    _instance = this;
  }

  factory RequestUtil([BaseOptions? options]) =>
      _instance ?? RequestUtil._internal(options);

  Future get(String path, [Map<String, dynamic>? params]) async {
    return await _dio?.get(path, queryParameters: params);
  }
}
```

## 工具使用

```dart 
void main(List<String> arguments) async {
  Map<String, dynamic> params = {
    "appid": "56761788",
    "appsecret": "ti3hP8y9",
    "city": "南京"
  };
  var res = await RequestUtil().get("/free/day", params);
  Map<String, dynamic> response = jsonDecode(res.toString());
  print("城市编号: ${response["cityid"]}");
  print("城市: ${response["city"]}");
  print("更新时间: ${response["update_time"]}");
  print("天气状况: ${response["wea"]}");
  print("平均温度: ${response["tem"]}");
  print("白天温度: ${response["tem_day"]}");
  print("晚上温度: ${response["tem_night"]}");
  print("风向: ${response["win"]}");
  print("等级: ${response["win_speed"]}");
  print("风速: ${response["win_meter"]}");
  print("空气质量: ${response["air"]}");
}
```

结果输出
```
城市编号: 101190101
城市: 南京
更新时间: 16:05
天气状况: 多云
平均温度: 23
白天温度: 24
晚上温度: 11
风向: 西风
等级: 2级
风速: 6km/h
空气质量: 70
```

上述只是简单封装了get请求，视具体业务增加其他接口封装。