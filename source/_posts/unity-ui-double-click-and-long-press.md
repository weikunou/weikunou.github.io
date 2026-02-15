---
title: Unity3D UI 双击和长按
date: 2024-11-10 18:41:50
tags: Unity3D
---

Unity3D 实现 UI 元素双击和长按功能。

<!--more-->

# UI 双击和长按

上一篇文章实现了拖拽接口，这篇文章来实现 UI 的双击和长按。

## 双击

创建脚本 `UIDoubleClick.cs`，创建一个 Image，并把脚本挂载到它身上。

在脚本中，继承 `IPointerClickHandler` 接口，实现 `OnPointerClick` 点击方法。

第一次点击时，记录点击的时间，如果第二次点击的时间，和上次点击时间的间隔非常短，则判定为双击。

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class UIDoubleClick : MonoBehaviour, IPointerClickHandler
{
    public float doubleClickThreshold = 0.2f; // 双击的时间间隔
    float lastClickTime = 0f; // 记录上次点击的时间

    public void OnPointerClick(PointerEventData eventData)
    {
        // 获取当前点击的时间
        float currentTime = Time.time;

        // 判断两次点击时间间隔是否在阈值范围内
        if (currentTime - lastClickTime < doubleClickThreshold)
        {
            OnDoubleClick();
        }

        // 更新上一次点击的时间
        lastClickTime = currentTime;
    }

    void OnDoubleClick()
    {
        Debug.Log("双击");
    }
}
```

运行效果：

![](../images/unity-ui-double-click-and-long-press/双击.gif)

## 长按

创建脚本 `UILongPress.cs`，并挂载到 Image 身上。

在脚本中，继承 `IPointerDownHandler` 和 `IPointerUpHandler` 接口，实现 `OnPointerDown`（按下）和 `OnPointerUp`（抬起）方法。

按下时，记录按下的时间和按住的状态，在 `Update` 中检查长按的时间和状态，达到长按的时间阈值后，执行一次长按的逻辑，并把长按状态重置。

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class UILongPress : MonoBehaviour, IPointerDownHandler, IPointerUpHandler
{
    public float longPressThreshold = 1.0f; // 长按的时间阈值
    float pressStartTime; // 按下的时间
    bool isPressing = false; // 是否按住

    public void OnPointerDown(PointerEventData eventData)
    {
        isPressing = true;
        pressStartTime = Time.time;
    }

    public void OnPointerUp(PointerEventData eventData)
    {
        isPressing = false;
    }

    void Update()
    {
        // 检查是否在长按状态
        if (isPressing && (Time.time - pressStartTime) > longPressThreshold)
        {
            OnLongPress();
            isPressing = false; // 只触发一次长按事件
        }
    }

    void OnLongPress()
    {
        Debug.Log("长按");
    }
}
```

运行效果：

![](../images/unity-ui-double-click-and-long-press/长按.gif)
