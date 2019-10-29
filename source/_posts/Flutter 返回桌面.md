---
title: Flutter 返回按钮返回桌面
date: 2019-10-29 21:42:23
tags: Golang
---
# Flutter 返回按钮返回桌面

想实现点击返回按钮，直接返回桌面，本想 flutter 有方法的话，直接用，然而好像没有，所以采用调用本地方法返回桌面

## Android 端 MainActivity 代码如下

```java
package com.dreamreal.example;

import android.os.Bundle;
import io.flutter.app.FlutterActivity;
import io.flutter.plugin.common.MethodChannel;
import io.flutter.plugins.GeneratedPluginRegistrant;

public class MainActivity extends FlutterActivity {
    // 字符串常量，回到手机桌面
    private final String chanel = "back/desktop";
    // 返回到桌面事件
    static final String backDesktopEvent = "backDesktop";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        GeneratedPluginRegistrant.registerWith(this);

        MethodChannel(getFlutterView(), chanel).setMethodCallHandler(
          (methodCall, result) -> {
            if (methodCall.method.equals(backDesktopEvent)) {
              moveTaskToBack(false);
              result.success(true);
            }
          }
        );
    }
  
}
```

如果是 kotlin 的话，如下：

```kotlin
package com.dreamreal.example

import android.os.Bundle

import io.flutter.app.FlutterActivity
import io.flutter.plugins.GeneratedPluginRegistrant
import io.flutter.plugin.common.MethodChannel

class MainActivity: FlutterActivity() {
  // 字符串常量，回到手机桌面
  private final val channel = "back/desktop";
  // 返回到桌面事件
  private final val backDesktopEvent = "backDesktop";

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    GeneratedPluginRegistrant.registerWith(this)

    MethodChannel(flutterView, channel).setMethodCallHandler { methodCall, result ->
      if (methodCall.method.equals(backDesktopEvent)) {
        moveTaskToBack(false);
        result.success(true);
      }
    }
  }
}

```



## Flutter 端代码如下

``` dart
import 'package:flutter/services.dart';
import 'package:flutter/material.dart';

class BackDesktop {
  // 字符串常量，回到手机桌面
  static const String chanel = "back/desktop";

  // 返回到桌面事件
  static const String backDesktopEvent = "backDesktop";

  // 返回到桌面方法
  static Future<bool> backDesktop() async {
    final platform = MethodChannel(chanel);
    try {
      await platform.invokeMethod(backDesktopEvent);
    } on PlatformException catch (e) {
      debugPrint(e.toString());
    }
    return Future.value(false);
  }
}
```



## 调用时如下

```dart
WillPopScope(
  onWillPop: BackDesktop.backDesktop, 
  child: 「这里是你需要拦截返回按钮的页面」
);
```

