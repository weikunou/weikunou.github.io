---
title: Unity3D Shader 顶点和片段着色器
date: 2025-12-07 11:38:42
tags: Unity3D
---

Unity3D Shader 顶点和片段着色器。

<!--more-->

# Unity3D Shader 顶点和片段着色器

Unity 的自定义 Shader 一般由两部分最核心的程序组成：

- **顶点着色器（Vertex Shader，简称 vert）**
- **片段着色器（Fragment Shader，简称 frag）**

如果你刚入门 Shader，这两个函数是你最先要理解的东西，因为所有渲染行为都从这里开始。

## 什么是顶点着色器（vert）

顶点着色器（vertex shader）负责“**处理网格的每一个顶点**”。

Unity Mesh 的每个点（MeshFilter 里的那个 Mesh）都会被送入顶点着色器，Shader 内的 `appdata` 就是顶点的结构体。

它主要负责：

### 将对象空间坐标转换到裁剪空间

也就是：

- 本地坐标 → 世界坐标
- 世界坐标 → 视图空间（Camera）
- 视图空间 → 裁剪空间（Clip Space）

最终输出给 GPU 进行光栅化。

在 Unity 中通常使用：

```c
UnityObjectToClipPos(v.vertex)
```

### 传递数据给片段着色器（颜色 / UV / 法线等）

顶点着色器可以将数据装进 `v2f`（vertex-to-fragment）结构体中，传递到片段着色器。

例如传递 UV：

```c
o.uv = v.uv;
```

### 可以修改顶点位置（顶点动画）

例如：

- 顶点扭曲
- 呼吸效果
- 旗帜随风摆动
- 水波

顶点动画就是在这里实现的。

## 什么是片段着色器（frag）

片段着色器（fragment shader）负责“**处理每个像素最后显示什么颜色**”。

它的输入是经过光栅化之后的“片段”（像素点），每个 Pixel 都会跑一次 `frag()`。

片段着色器通常负责：

### 计算最终颜色

包含：

- 采样贴图（Texture）
- 颜色混合
- Light / BRDF
- 各类特效（溶解、描边、灰度化、等）

例如返回一个固定颜色：

```c
return fixed4(1, 0, 0, 1); // 红色
```

### 采样纹理

```c
fixed4 col = tex2D(_MainTex, i.uv);
```

### 片段着色器比顶点着色器更耗性能

因为屏幕可能有 **百万级像素**，每个像素都要跑 `frag`，所以：

- 顶点越少 = 越省性能
- 片段越复杂 = 越吃性能

如果你做的是 UI、全屏特效、后处理，更要注意 fragment 的复杂度。

# 最基础的 vert / frag Shader

下面代码是 Unity Shader 中最基础的顶点 + 片段结构。只做两件事：

- `vert`：把顶点从模型空间变成裁剪空间
- `frag`：返回 `_Color` 颜色作为像素颜色

```c
Shader "Custom/VertFragDemo"
{
    Properties
    {
        _Color("Main Color", Color) = (1,1,1,1)
    }

    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
            };

            fixed4 _Color;

            v2f vert (appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return _Color;
            }
            ENDCG
        }
    }
}
```

# 可视化理解：顶点着色 → 光栅化 → 片段着色

以下是 GPU 渲染的基本流程，非常重要：

```c
 Mesh 顶点数据
       ↓
 顶点着色器（vert）
       ↓
 裁剪、组装三角形
       ↓
 光栅化（生成像素）
       ↓
 片段着色器（frag）
       ↓
 Framebuffer（最终图像）
```

你可以看到：

- **vert 运行次数：顶点数量**（通常比较少）
- **frag 运行次数：像素数量**（可能是百万级）

所以片段着色器比顶点着色器更重要、也更昂贵。

# 常用的 appdata / v2f 字段

## appdata（从 Mesh 输入）

```c
struct appdata
{
    float4 vertex : POSITION;
    float2 uv     : TEXCOORD0;
    float3 normal : NORMAL;
};
```

## v2f（传递给 frag）

```c
struct v2f
{
    float4 pos : SV_POSITION;
    float2 uv  : TEXCOORD0;
};
```

你想传什么给 frag 就在这里声明。

# 实战：加一点变化（UV 贴图 + 色调）

下面是一个同时使用顶点 + 片段 + 贴图采样的版本：

```c
Shader "Custom/TextureTint"
{
    Properties
    {
        _MainTex("Texture", 2D) = "white" {}
        _Tint("Tint Color", Color) = (1,1,1,1)
    }

    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 _Tint;

            v2f vert(appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
                return col * _Tint;
            }
            ENDCG
        }
    }
}
```

你可以：

- 给模型贴一张纹理
- 用 `_Tint` 调节颜色

这是一个最常用的 vert/frag 模板，几乎所有 Unity 自定义 Shader 都从这里开始。

# 总结

| 内容     | 顶点着色器（vert）           | 片段着色器（frag）         |
| -------- | ---------------------------- | -------------------------- |
| 输入     | Mesh 顶点数据                | 每个像素                   |
| 功能     | 坐标变换、传递数据、顶点动画 | 计算最终颜色、采样纹理     |
| 运行次数 | 顶点数量                     | 像素数量（大量）           |
| 性能影响 | 小                           | 大                         |
| 典型用途 | 旗帜飘动、水波、形变         | 灯光、贴图、特效、颜色计算 |
