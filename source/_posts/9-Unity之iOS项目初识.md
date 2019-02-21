---
title: Unity之iOS项目初识
date: 2019-02-21 20:55:13
categories: Unity开发
tags: [Unity,iOS]
---
## 前言
Unity 是一款跨平台2D / 3D 游戏引擎，作为 Unity 开发者，我们有必要了解一些iOS平台的相关知识。<!-- more -->

## 工具准备
Unity、Xcode

## 导出项目
因为 iOS 项目比较特殊，Unity只能导出项目然后才能在 Xcode 运行、打包以及上传
- 选择平台 打开 File/Build Settings

<img src="select_platform.png" width = "260" height = "331"/>
<!-- ![select_platform](select_platform.png) -->

- 选择SDK 打开 Player Settings，找到Target SDK

<!-- ![select_sdk](select_sdk.png) -->
<img src="select_sdk.png" width = "400" height = "478" />
>  **Device SDK** 是在真机上运行，**Simulator SDK** 是在模拟器上运行。

- 选择好SDK之后，直接点击 Build，导出的工程目录结构如下

![project_dir](project_dir.png)
> 双击 **Unity-iPhone.xcodeproj** 可以直接打开Xcode运行工程

***注：*** 导出项目本身不需要任何 iOS 相关的环境，因此使用 **Windows** 也可以导出 iOS 项目

## Unity 与 iOS 交互
### Unity 调用 iOS
#### 原理
- 1、Unity支持 C# 通过调用DLL的方式来调用 C++
- 2、iOS支持 C++ 和 OC混编的， C++ 可以调用 OC
> 基于以上两点，我们可以通过 C# -> C++ -> OC实现 Unity 调用 OC

#### Unity 端代码
- 引入头文件

```CSharp
using System.Runtime.InteropServices;
```
- 声明

```CSharp
[DllImport("__Internal")]
private static extern string Func1Bridge(int pack);

public struct strutDict
{
    public string key;
    public string value;
}
[DllImport("__Internal")]
private static extern void Func2Bridge(strutDict dict);
```

- 调用

```CSharp
public void btnClick(int key)
{
    switch (key)
    {
        case 0:
#if UNITY_IOS && !UNITY_EDITOR
            string str = Func1Bridge(1);
            Toast.instance.show(str);
#endif
            break;
        case 1:
            strutDict strutDict = new strutDict();
            strutDict.key = "key1";
            strutDict.value = "value1";
#if UNITY_IOS && !UNITY_EDITOR
            Func2Bridge(strutDict);
#endif
            break;
    }
}
```

#### iOS 端代码
```C
extern "C" {
    char* Func1Bridge(int pack) {
        NSLog(@"Func1Bridge:%d",pack);
        return strdup([@"iOS click" UTF8String]);
    }

    struct strutDict
    {
         const char* key;
         const char*  value;
    };
    void Func2Bridge(strutDict dict) {
        NSLog(@"Func2Bridge---key:%@,value:%@",
        [NSString stringWithUTF8String:dict.key],
        [NSString stringWithUTF8String:dict.value]);
    }
}
```

> **最后，经过尝试**
>- Unity向iOS传递参数类型：bool、int、float、string(char \*)、struct 等
>- 调用后的返回值：bool、int、float、string(char \*) 等

### iOS 调用 Unity
```C
UnitySendMessage("GameObjectName", "MethodName", "Message");
```
> iOS 调用 Unity 方法比较简单，传递的参数也只有string类型，并且只能传递一个，没有返回值

## 更新 Unity 端代码
当 Untiy 代码有更新之后，需要重新打包导出项目，此时旧项目已经有了 iOS 端代码，直接覆盖显然不合适，只需要替换部分文件即可。
> 需要替换的文件为 **MapFileParser**， **MapFileParser.sh**，**data**，**Libraries**，**Classes/Native**

![changed_dir](changed_dir.png)

> **注意**
>- 1、Native 文件夹直接替换有时会出现问题，最好是在Xcode里面删除，然后再引用最新的
>- 2、如果 Unity 代码调用到平台硬件（相机、麦克风等）运行到设备之后无法使用硬件，需要替换整个项目，然后重新导入iOS代码。

## 拓展
### P/Invoke回调函数
#### Unity 端代码
- 声明  

```CSharp
public delegate void LibCallback(string msg);
[AOT.MonoPInvokeCallback(typeof(LibCallback))]
public static void Callback(string msg)
{
    Toast.instance.show(msg);
}

[DllImport("__Internal")]
public static extern void SetLibCallback(LibCallback callback);
```

- 设置回调

```CSharp
void Start () {
      Debug.Log("Start");
#if UNITY_IOS && !UNITY_EDITOR
      SetLibCallback(Callback);
#endif
}
```
#### iOS 端代码
- 声明

```C
typedef void(*monoCallback)(const char* str);
extern "C" {
    typedef void(*monoCallback)(const char* str);
    void SetLibCallback(monoCallback callback);
    void CallTest();
}

class MonoHandler {
private:
    monoCallback m_callback;

public:
    void setCallback(monoCallback callback) {
        m_callback = callback;
    }

    void sendMesage(const char* msg) {
        m_callback(msg);
    }
};

MonoHandler handler;

void SetLibCallback(monoCallback callback) {
    handler.setCallback(callback);
}

void CallTest() {
    const char* p = "message from lib";
    handler.sendMesage(p);
}
```

- 调用

```C
void Func2Bridge(strutDict dict) {
    CallTest();
}
```

## 总结
本文主要分享了 Unity 与 iOS 之间的交互，以及Unity导出 iOS 工程与更新到最新代码的方法。

### 参考博客
[Unity官方文档](https://docs.unity3d.com/Manual/PluginsForIOS.html)

[与iOS、Android的交互 理论篇](https://www.jianshu.com/p/35ade1078d79)
