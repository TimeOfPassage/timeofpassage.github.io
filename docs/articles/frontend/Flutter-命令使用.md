# Flutter-命令使用

## 1. 环境安装

参考flutter中文网[安装教程](https://flutter.cn/docs/get-started/install)，根据不同环境进行安装，推荐编辑`xscode`

使用命令`flutter doctor`进行环境检测


## 2. 项目创建

1. 基本创建命令如下

> flutter create xxx

**tips: xxx表示项目名**

2. 如果要置顶语言：使用`-i`指定iOS语言，`-a`指定Android语言

> flutter create -i swift -a kotlin xxx

3. 使用`-org`替代`com.example`

> flutter create --org com.txzhe xxx

4. 使用`--help`查看帮助信息

> flutter create --help

```bash
Usage: flutter create <output directory>
-h, --help                     Print this usage information.
    --platforms                the platforms supported by this project.This argument only works when the --template is set to app or plugin.Platform folders (e.g. android/) will be generated in the target project.When adding platforms to a plugin project, the
                               pubspec.yaml will be updated with the requested platform.Adding desktop platforms requires the corresponding desktop config setting to be enabled.
                               [ios (default), android (default), windows (default), linux (default), macos (default), web (default)]
    --[no-]pub                 Whether to run "flutter pub get" after the project has been created.
                               (defaults to on)
    --[no-]offline             When "flutter pub get" is run by the create command, this indicates whether to run it in offline mode or not. In offline mode, it will need to have all dependencies already available in the pub cache to succeed.
    --[no-]with-driver-test    Also add a flutter_driver dependency and generate a sample 'flutter drive' test.
-t, --template=<type>          Specify the type of project to create.

          [app]                (default) Generate a Flutter application.
          [module]             Generate a project to add a Flutter module to an existing Android or iOS application.
          [package]            Generate a shareable Flutter project containing modular Dart code.
          [plugin]             Generate a shareable Flutter project containing an API in Dart code with a platform-specific implementation for Android, for iOS code, or for both.

-s, --sample=<id>              Specifies the Flutter code sample to use as the main.dart for an application. Implies --template=app. The value should be the sample ID of the desired sample from the API documentation website (http://docs.flutter.dev). An example
                               can be found at https://master-api.flutter.dev/flutter/widgets/SingleChildScrollView-class.html
    --list-samples=<path>      Specifies a JSON output file for a listing of Flutter code samples that can be created with --sample.
    --[no-]overwrite           When performing operations, overwrite existing files.
    --description              The description to use for your new Flutter project. This string ends up in the pubspec.yaml file.
                               (defaults to "A new Flutter project.")
    --org                      The organization responsible for your new Flutter project, in reverse domain name notation. This string is used in Java package names and as prefix in the iOS bundle identifier.
                               (defaults to "com.example")
    --project-name             The project name for this new Flutter project. This must be a valid dart package name.
-i, --ios-language             [objc, swift (default)]
-a, --android-language         [java, kotlin (default)]
```