---
title: Unity3D 截图
date: 2024-11-24 15:34:23
tags: Unity3D
---

使用 Unity3D 自带的截图接口，制作截图工具。

<!--more-->

# 截图

有时候我们想对 Unity 的窗口进行截图，如果直接使用一些截图工具，很难截取到一张完整分辨率的图片（例如，我们想要截取一张 1920 * 1080 的图片）。

其实 Unity 有提供截图的接口，我们只需要写一个脚本，把截图接口做成简单的菜单栏工具即可。

## 创建工具脚本

创建脚本 `ScreenshotTool.cs`，写一个 `CaptureFull` 方法，调用 Unity 提供的 `ScreenCapture.CaptureScreenshot` 方法即可。

截图时，为了方便找到，保存的路径是 Assets 文件夹（`Application.dataPath`），截图的名称是 Screenshot 拼接了当前的时间（如果名称一样，每次截图都会覆盖原来的图片）。

我们在 `CaptureFull` 方法上面添加一个 `MenuItem`，就可以在菜单栏找到它，也可以使用 Alt + Q 快捷键（即路径末尾的 `&Q`）。

```csharp
using System;
using UnityEngine;
using UnityEditor;

public static class ScreenshotTool
{
    [MenuItem("截图/截取全屏 &Q")]
    public static void CaptureFull()
    {
        string time = DateTime.Now.ToString("yyyy-MM-dd_HH-mm-ss");
        string path = $"{Application.dataPath}/Screenshot_{time}.png";
        ScreenCapture.CaptureScreenshot(path);
        Debug.Log("Screenshot saved at: " + path);
    }
}
```

## 截图效果

如图，菜单栏出现按钮，并且有快捷键的描述。

运行游戏时，点击菜单栏按钮，或者按下 Alt + Q，控制台会打印截图的保存路径。

当 Unity 资源文件夹刷新时，也会看到 Assets 文件夹下多出来一张图片，尺寸是 1920 * 1080（当前 Game 窗口的分辨率）。

![](../images/unity-screenshot-tool/截图效果.png)
