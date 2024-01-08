# #Flutter单例实现

# 懒汉式

```dart
/// 懒汉式
class Singleton {
  static Singleton _instance;

  Singleton._internal() {
    _instance = this;
  }

  factory Singleton() => _instance ?? Singleton._internal();
}
```

# 饿汉式

```dart
/// 饿汉式
class Singleton {
  Singleton._internal();
  
  factory Singleton() => _instance;
  
  static late final Singleton _instance = Singleton._internal();
}
```
