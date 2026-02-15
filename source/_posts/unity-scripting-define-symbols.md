---
title: Unity3D 平台宏定义
date: 2025-07-27 10:37:27
tags: Unity3D
---

Unity3D 平台宏定义的介绍和使用。

<!--more-->

# 平台宏定义

在 Unity 开发过程中，经常需要写一些平台相关、调试用、或只在编辑器运行的代码。

为了避免打包错误或平台兼容问题，我们可以使用 Unity 提供的 **宏指令（平台宏定义）** 来控制代码的编译范围。

一个最常见的示例如下：

```csharp
#if UNITY_EDITOR
    Debug.Log("仅在编辑器中输出");
#endif
```

这段代码只会在 Unity 编辑器中编译执行，打包时会被自动忽略。

## 常见 Unity 平台宏定义一览

| 宏定义                 | 说明                                    |
| ---------------------- | --------------------------------------- |
| `UNITY_EDITOR`         | Unity 编辑器中生效                      |
| `UNITY_ANDROID`        | 安卓平台                                |
| `UNITY_IOS`            | iOS 平台                                |
| `UNITY_STANDALONE_WIN` | Windows 平台（PC 端）                   |
| `UNITY_STANDALONE_OSX` | macOS 平台                              |
| `UNITY_STANDALONE`     | 所有桌面平台（Windows、Mac、Linux）通用 |
| `UNITY_WEBGL`          | WebGL 平台                              |
| `UNITY_SERVER`         | Server 构建模式（无图形）               |
| `DEVELOPMENT_BUILD`    | 开发者模式构建                          |
| `UNITY_EDITOR_WIN`     | Windows 编辑器环境                      |
| `UNITY_EDITOR_OSX`     | macOS 编辑器环境                        |

> 💡 Unity 会根据打包平台自动启用这些宏，无需手动添加，直接在代码中使用即可。

## 实际使用示例

### 编辑器专用代码

如菜单功能、调试工具等。

```csharp
#if UNITY_EDITOR
using UnityEditor;
#endif

public class EditorOnlyTool
{
    public void StopPlaymode()
    {
#if UNITY_EDITOR
        EditorApplication.isPlaying = false;
#endif
    }
}
```

### 路径适配

适用于不同平台存档路径处理。

```csharp
string GetSavePath()
{
#if UNITY_EDITOR
    return Application.dataPath + "/";
#else
    return Application.persistentDataPath + "/";
#endif
}
```

## 自定义宏定义

除了系统默认的宏，我们也可以在项目中自定义宏。

设置位置：<kbd>Edit</kbd> > <kbd>Project Settings</kbd> > <kbd>Player</kbd> > <kbd>Other Settings</kbd> > <kbd>Scripting Define Symbols</kbd>

> 💡 自定义宏非常适合用于功能开关、调试模式、模块切换等场景。

### 小游戏平台

例如添加：

```csharp
WX_GAME;DY_GAME;
```

然后代码中使用：

```csharp
#if WX_GAME
    // 微信小游戏逻辑
#elif DY_GAME
    // 抖音小游戏逻辑
#else
    // 其他平台逻辑
#endif
```

在打包微信小游戏时，移除 `DY_GAME` 宏定义，只会编译微信小游戏的逻辑。同理，打包抖音小游戏时，移除 `WX_GAME` 宏定义。

可以写一个菜单工具，一键切换平台宏定义，避免手动删来删去。

```csharp
using System.Linq;
using UnityEditor;
using UnityEngine;

public static class PlatformMacroSwitcher
{
    // 所有小游戏平台的宏，用于互斥
    private static readonly string[] MiniGameMacros = { "WX_GAME", "DY_GAME" };

    [MenuItem("Tools/小游戏平台切换/切换到微信小游戏")]
    public static void SwitchToWXGame()
    {
        SetExclusiveMacro("WX_GAME");
    }

    [MenuItem("Tools/小游戏平台切换/切换到抖音小游戏")]
    public static void SwitchToDYGame()
    {
        SetExclusiveMacro("DY_GAME");
    }

    private static void SetExclusiveMacro(string targetMacro)
    {
        var targetGroup = EditorUserBuildSettings.selectedBuildTargetGroup;
        var symbols = PlayerSettings.GetScriptingDefineSymbolsForGroup(targetGroup);
        var symbolList = symbols.Split(';').Where(s => !string.IsNullOrWhiteSpace(s)).ToList();

        // 移除所有小游戏平台宏
        symbolList.RemoveAll(s => MiniGameMacros.Contains(s));

        // 添加目标平台宏
        if (!symbolList.Contains(targetMacro))
        {
            symbolList.Add(targetMacro);
        }

        // 更新宏定义
        string newSymbols = string.Join(";", symbolList);
        PlayerSettings.SetScriptingDefineSymbolsForGroup(targetGroup, newSymbols);

        Debug.Log($"已切换到平台宏：{targetMacro}");
    }
}
```

### 日志开关

例如添加：

```csharp
ENABLE_LOG;
```

封装一个日志工具类，可以通过宏定义控制是否打印日志。

```csharp
using UnityEngine;

public static class LogUtil
{
    [System.Diagnostics.Conditional("ENABLE_LOG")]
    public static void Log(string message)
    {
        Debug.Log(message);
    }

    [System.Diagnostics.Conditional("ENABLE_LOG")]
    public static void LogWarning(string message)
    {
        Debug.LogWarning(message);
    }

    [System.Diagnostics.Conditional("ENABLE_LOG")]
    public static void LogError(string message)
    {
        Debug.LogError(message);
    }
}
```

可以写一个菜单工具，一键开关日志宏定义，避免手动删来删去。

```csharp
using System.Linq;
using UnityEditor;
using UnityEngine;

public static class ToggleLogMacro
{
    [MenuItem("Tools/日志开关/开启 ENABLE_LOG")]
    public static void EnableLog()
    {
        ToggleMacro("ENABLE_LOG", true);
    }

    [MenuItem("Tools/日志开关/关闭 ENABLE_LOG")]
    public static void DisableLog()
    {
        ToggleMacro("ENABLE_LOG", false);
    }

    private static void ToggleMacro(string macro, bool enable)
    {
        var targetGroup = EditorUserBuildSettings.selectedBuildTargetGroup;
        var symbols = PlayerSettings.GetScriptingDefineSymbolsForGroup(targetGroup);
        var symbolList = symbols.Split(';');

        if (enable && !symbols.Contains(macro))
        {
            symbols = string.IsNullOrEmpty(symbols) ? macro : symbols + ";" + macro;
            Debug.Log($"启用宏定义: {macro}");
        }
        else if (!enable && symbols.Contains(macro))
        {
            symbols = string.Join(";", symbolList.Where(s => s != macro));
            Debug.Log($"关闭宏定义: {macro}");
        }

        PlayerSettings.SetScriptingDefineSymbolsForGroup(targetGroup, symbols);
    }
}
```

## 使用技巧与注意事项

### 建议使用的场景

- ✅ **平台适配**：Android/iOS 的不同 API、路径、权限处理
- ✅ **调试工具**：只在开发时使用的代码
- ✅ **构建优化**：排除测试代码、防止泄露敏感信息

### 不建议滥用宏指令

- ❌ 大段平台逻辑堆叠在一个文件中，可读性差
- ❌ 宏代替运行时判断，导致运行逻辑不一致
- ❌ 自定义宏未文档化，易被遗忘或误用

## 结语

Unity 的宏指令是跨平台开发的重要工具，它让我们可以为不同平台编写不同的逻辑代码，也能帮助我们控制调试和发布行为。

但使用时要保持 **清晰结构** 和 **文档记录** ，避免维护困难。
