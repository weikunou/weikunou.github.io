---
title: Unity3D Shader 属性详解
date: 2025-10-26 16:47:55
tags: Unity3D
---

Unity3D Shader 属性详解。

<!--more-->

# Unity3D Shader 属性详解

在 Shader 里，我们经常能看到这样的代码片段：

```c
Properties
{
    _Color ("Main Color", Color) = (1,1,1,1)
    _MainTex ("Base Texture", 2D) = "white" {}
}
```

这些写在 `Properties {}` 里的内容，就是 **Shader 属性（Properties）**。

它们的作用是让 **美术或程序** 能够在 **Unity 材质面板** 或 **C# 脚本** 中修改 Shader 内部变量，从而让同一个 Shader 产生不同的视觉效果。

## Properties 是什么

`Properties` 块定义的是 **Shader 可以从外部编辑的参数**。

这些参数会自动出现在材质球（Material）面板上。

在 `SubShader` 中，你会用到同名变量（例如 `_Color`、`_MainTex`），Unity 会自动把它们链接起来。

```c
Properties
{
    _Color ("Main Color", Color) = (1,1,1,1)
}

SubShader
{
    fixed4 _Color; // 对应 Properties 中的 _Color
}
```

## Properties 的常见类型

| 类型       | 示例                                        | 默认值格式 | 面板表现       |
| ---------- | ------------------------------------------- | ---------- | -------------- |
| **Color**  | `_Color("Main Color", Color) = (1,1,1,1)`   | RGBA       | 颜色选择器     |
| **Vector** | `_Offset("Offset", Vector) = (1,0,0,0)`     | 4D 向量    | 四个浮点输入框 |
| **Range**  | `_Intensity("Intensity", Range(0,1)) = 0.5` | 数值区间   | 滑动条         |
| **Float**  | `_Power("Power", Float) = 2.0`              | 单个数值   | 数值输入框     |
| **2D**     | `_MainTex("Base Texture", 2D) = "white" {}` | 贴图       | 纹理选择框     |
| **Cube**   | `_CubeMap("Environment", Cube) = "" {}`     | 立方体贴图 | CubeMap 选择   |
| **3D**     | `_VolumeTex("Volume", 3D) = "black" {}`     | 三维贴图   | Volume 选择    |

## Properties 与 Shader 变量的对应关系

定义完 Properties 后，要在 CG 代码里定义同名变量，类型要匹配：

| Properties 类型 | CG/HLSL 变量类型           |
| --------------- | -------------------------- |
| Color / Vector  | `float4` / `fixed4`        |
| Range / Float   | `float` / `half` / `fixed` |
| 2D              | `sampler2D`                |
| Cube            | `samplerCUBE`              |
| 3D              | `sampler3D`                |

例如：

```c
Properties
{
    _Color ("Main Color", Color) = (1,1,1,1)
    _MainTex ("Base Texture", 2D) = "white" {}
}

SubShader
{
    fixed4 _Color;
    sampler2D _MainTex;
}
```

## C# 中修改 Shader 属性

在脚本中可以通过材质访问这些属性：

```csharp
Renderer renderer = GetComponent<Renderer>();
Material mat = renderer.material;

// 设置颜色
mat.SetColor("_Color", Color.red);

// 设置纹理
mat.SetTexture("_MainTex", someTexture);

// 获取浮点数
float power = mat.GetFloat("_Intensity");
```

> 注意：属性名要与 Shader 中的变量名（如 `_Color`）**完全一致**。

## 默认贴图关键字

Unity 内置了一些特殊的默认贴图关键字，方便调试：

| 默认字符串 | 说明             |
| ---------- | ---------------- |
| `"white"`  | 白色纹理         |
| `"black"`  | 黑色纹理         |
| `"gray"`   | 灰色纹理         |
| `"bump"`   | 法线贴图默认纹理 |
| `""`       | 空贴图           |

例如：

```c
_MainTex ("Texture", 2D) = "white" {}
```

表示如果没有指定纹理，就使用一张白色默认贴图。

## Range 与 Float 的区别

`Range` 只是带有一个 UI 滑条的 `Float`。

两者在 Shader 代码里其实是一样的类型。

```c
_RangeValue ("Brightness", Range(0, 2)) = 1
_FloatValue ("Brightness", Float) = 1
```

在代码中它们都对应一个 `float`，区别仅在于 Inspector 显示方式。

## 隐藏属性与调试技巧

有时你不希望某个属性在面板中显示，可以用 `[HideInInspector]`：

```c
[HideInInspector]_HiddenValue ("Hidden", Float) = 1
```

其他常用标签：

| 修饰符            | 说明                          |
| ----------------- | ----------------------------- |
| `[NoScaleOffset]` | 关闭纹理的 Tiling/Offset 控制 |
| `[Normal]`        | 表示贴图是法线贴图            |
| `[HDR]`           | 支持高动态范围颜色            |
| `[Toggle]`        | 显示成开关                    |
| `[Enum]`          | 枚举选择                      |
