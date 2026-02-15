---
title: Unity3D Shader 结构详解
date: 2025-11-23 10:35:40
tags: Unity3D
---

Unity3D Shader 结构详解。

<!--more-->

# Unity3D Shader 结构详解

当你打开一个 Unity Shader 文件时，第一眼看到的一定是这样的结构：

```c
Shader "Custom/Simple"
{
    Properties { ... }

    SubShader
    {
        Pass { ... }
    }
}
```

刚接触 Shader 时，你可能会疑惑：

- **SubShader 是什么？为什么有多个？**
- **Pass 是做什么的？能不能只写一个？**
- **为什么有些 Shader 会写 UsePass 或 GrabPass？**
- **Unity 渲染时到底会用哪个 SubShader？**

本篇将从宏观结构入手，带你彻底理解 Shader 的层级与执行顺序。

# Shader 的总体结构

Unity Shader 最顶层结构如下：

```c
Shader
 ├── Properties
 ├── SubShader（通常 1~2 个）
 │     ├── Tags
 │     ├── Pass（一个或多个）
 │     └── UsePass / GrabPass（可选）
 └── Fallback（可选）
```

你可以把结构理解为：

- **Shader 是一本书**
- **SubShader 是书的不同版本（兼容方案）**
- **Pass 是每个版本中的章节（渲染步骤）**

## Properties 外部可调节的属性

一个 Shader 可以定义外部可编辑的属性：

```c
Properties
{
    _Color("Color", Color) = (1,1,1,1)
}
```

这些属性会自动出现在材质面板(Mesh Renderer → Material)。

我们在上一篇《Unity3D Shader 属性详解》已经详细讲过，这里略过。

## SubShader 多个兼容方案（Unity 自动选择）

一个 Unity Shader 可以包含 **多个 SubShader**。

示例：

```c
SubShader { ... }  // 高级显卡使用
SubShader { ... }  // 低级显卡使用
FallBack "Diffuse" // 都不支持则使用 Fallback
```

Unity 的选择规则：

Unity **从上到下依次测试** SubShader，遇到 **支持的平台能力** 的那一个就使用它。

例如：

| SubShader        | 说明                                              |
| ---------------- | ------------------------------------------------- |
| 第一个 SubShader | 用到了高端特性（例如 Shader Model 4.5、曲面细分） |
| 第二个 SubShader | 使用了更古老、低端显卡支持的语法                  |

在手机端，Unity 会跳过无法执行的 SubShader，改用第二个。

你可以把 SubShader 理解为

- PC 端 → 高画质版本
- 手机端 → 简化版本
- 老设备 → 最低画质版本

## Pass 渲染步骤（每个 Pass 画一遍）

每个 SubShader 可以包含多个 Pass：

```c
SubShader
{
    Pass { ... }  // 第一次渲染
    Pass { ... }  // 第二次渲染
}
```

**一个 Pass = 一次 DrawCall**

多 Pass Shader 常用于：

- 描边 Outline（Pass1 渲染外轮廓，Pass2 渲染本体）
- 闪光效果（先渲染辉光，再渲染本体）
- 延迟渲染（分多步输出数据）
- 阴影渲染（ShadowCaster、ShadowCollector）

### 最简单的单 Pass Shader 如

```c
Pass
{
    CGPROGRAM
    #pragma vertex vert
    #pragma fragment frag
    ENDCG
}
```

### 多 Pass Shader 示例：简单角色描边

```c
SubShader
{
    // Pass1：渲染外描边
    Pass
    {
        // 外扩、单色渲染
    }

    // Pass2：渲染物体本体
    Pass
    {
        // 正常贴图渲染
    }
}
```

最终输出效果是两个 Pass 合成的结果。

## Tags 渲染排序与类别的重要设置

SubShader 和 Pass 都可以设置 Tags。

SubShader 常用 Tags

```c
Tags { "RenderType"="Opaque" "Queue"="Geometry" }
```

常见值：

| RenderType        | 说明       |
| ----------------- | ---------- |
| Opaque            | 不透明物体 |
| Transparent       | 透明物体   |
| TransparentCutout | 透明裁剪   |
| Background        | 背景       |
| Overlay           | UI、特效等 |

Queue（渲染队列）决定渲染顺序：

| 队列        | 数值 | 说明               |
| ----------- | ---- | ------------------ |
| Background  | 1000 | 最早绘制           |
| Geometry    | 2000 | 不透明物体         |
| AlphaTest   | 2450 | 透明裁剪           |
| Transparent | 3000 | 半透明物体（晚画） |
| Overlay     | 4000 | UGUI 最后画        |

半透明一定要排在后面，否则会穿插错误。

## UsePass 复用其他 Shader 的 Pass

可引用其他 Shader 的 Pass：

```c
UsePass "Legacy Shaders/Specular/BASE"
```

用途：

- 自己写一个 Pass，其他部分继承内置 Shader
- 避免重复代码
- 快速拼装效果

## GrabPass 抓取屏幕（用于玻璃、水面等特效）

示例：

```c
GrabPass {}
```

GrabPass 会把当前屏幕渲染成一张纹理，让 Shader 可以使用：

常用于：

- 水波扰动
- 玻璃折射
- 高斯模糊
- UI 变形效果

但要注意：

**GrabPass 很耗性能**，移动端慎用。

## Fallback 多数新项目用不到

```c
Fallback "Diffuse"
```

意味着：

如果所有 SubShader 都不支持，则使用 Unity 内置的 Diffuse。

现代项目中通常不使用 Fallback，因为我们会自己写多个 SubShader 兼容。

# 总结

| 概念           | 作用                 | 重点                       |
| -------------- | -------------------- | -------------------------- |
| **Shader**     | 顶层容器             | 同一个 Shader 对应一个材质 |
| **Properties** | 外部可调参数         | 可在 Inspector 和 C# 调节  |
| **SubShader**  | 不同设备的兼容版本   | Unity 自动选择支持的版本   |
| **Pass**       | 一次实际渲染         | 多 Pass = 多次 DrawCall    |
| **Tags**       | 渲染顺序/类型        | Queue 和 RenderType 最重要 |
| **UsePass**    | 复用别的 Pass        | 对内置 Shader 的继承       |
| **GrabPass**   | 抓屏                 | 性能消耗大，慎用           |
| **Fallback**   | 无法渲染时的后备方案 | 新项目一般不用             |

> **Shader = 属性 + 若干 SubShader；**
>
> **SubShader = 若干 Pass；**
>
> **Pass 就是一次画图，本质是一个 DrawCall。**
