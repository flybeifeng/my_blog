---
title: SwiftUI学习笔记
date: 2020-05-30 15:15:38
categories: iOS开发
tags: [iOS]
---
## 前言
SwiftUI 是苹果在2019年 WWDC 大会推出基于 Swift 语言构建的全新 UI 框架，与近年来大热的 React 和 Flutter 一样，都是采取了声明式编程。学习 SwiftUI 可以让我们了解最新的编程思想与技术。<!-- more -->

## SceneDelegate
从 Xcode11 发布以来，当你使用新 XCode 创建一个新的 iOS 项目时，SceneDelegate 会被默认创建，SceneDelegate 将负责 AppDelegate 的某些功能，在此不做额外的解释，有兴趣的可以自行搜索。在这里，我们主要说明 SceneDelegate 在 SwiftUI 项目中如何使用。
##### 1、配置Info.plist
![SwiftUI_SceneDelegate](SwiftUI_SceneDelegate.png)

##### 2、初始化代码
```Swift

func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {

    let contentView = ContentView()

    if let windowScene = scene as? UIWindowScene {
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = UIHostingController(rootView: contentView)
        self.window = window
        window.makeKeyAndVisible()
    }
}

```

## Hello World
```Swift
struct FirstView : View {
    var body: some View {
        Text("Hello World!")
    }
}
```
#### 实时预览
![hello_preview](hello_preview.png)

## 组件
上面是 SwiftUI 的一个简单演示，要真正在开发中使用 SwiftUI，还需我们掌握与熟练使用各种类型的组件。以下我们简单介绍几种常见的组件。

##### 基础组件
+ `Text` 用来显示文字 类似于 UIKit 中的 `UILabel`

+ `Image` 用来显示图片 类似于 UIKit 中的 `UIImageView`

+ `Spacer` 用来填充空白

##### 布局组件
+ `VStack` 竖直摆放的组合组件

+ `HStack` 水平摆放的组合组件

+ `List` 用来展示列表 类似于 UIKit 中的`UITableView`

![FirstView](FirstView.png)

##### 功能型组件
+ `NavigationView` 展示导航栏 类似于 `UINavigationBar`
+ `NavigationLink` 类似于`pushViewController:`方法

![ContentView](ContentView.png)

以上只是列出几种常见的组件，想要了解更多组件可以查看[苹果官网](https://developer.apple.com/documentation/swiftui/)

#### 演示
![demo](demo.gif)

## 最后
经过简单的演示，我们已经看到了 SwiftUI 的简洁与便利性。后续，SwiftUI 还可能支持跨平台，赶紧学习起来！
