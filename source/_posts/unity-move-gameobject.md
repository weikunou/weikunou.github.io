---
title: Unity3D 移动游戏物体
date: 2023-09-12 14:28:04
tags: Unity3D
---

在 unity 中，一个很基础的需求是移动游戏物体。

移动物体的方式有很多种，本质上都是对 Transform 组件的 Position 属性进行修改。

<!--more-->

## 一、直接修改 position

修改自身的位置：

```c#
transform.position = new Vector3(1, 0, 0);
```

当前物体就会出现在指定的坐标位置。

修改引用物体的位置：

```c#
public Transform other;

void Start()
{
    other.position = new Vector3(1, 0, 0);
}
```



## 二、Translate

unity 提供了 transform.Translate() 函数，用于移动游戏物体。

下面的例子是让当前物体沿着 X 轴正方向持续移动。

```c#
public float speed = 5f;

void Update()
{
    transform.Translate(Vector3.right * speed * Time.deltaTime, Space.World);
}
```

Translate 函数的第一个参数，就是移动的方向和距离，使用一个向量作为输入。

可以看到，这里使用了物体的右方向（Vector3.right），定义了一个速度（speed），乘上增量时间修正（Time.deltaTime），作为最终的移动向量。

为什么要乘上 Time.deltaTime 呢？

这是因为 Update 函数每秒的执行次数是根据设备的性能来决定的。如果当前帧率是 60，那么 Update 函数每秒执行 60 次；如果帧率是 30，则执行 30 次。那么，物体的移动速度就不稳定了。

为了保持物体在不同帧率下，都是同样的移动速度，Time.deltaTime 作为一个变量，就发挥作用了。

- 在 60 帧的情况下，Time.deltaTime 的值为 1/60，Update 执行了 60 次，最终 1 秒移动了 Vector3.right * speed 的距离。
- 在 30 帧的情况下，Time.deltaTime 的值为 1/30，Update 执行了 30 次，最终 1 秒移动了 Vector3.right * speed 的距离。

第二个参数是坐标系，默认是使用物体自身的局部坐标系，也可以指定使用世界坐标系（Space.World）。



## 三、刚体

给物体挂上刚体组件（Rigidbody），修改 velocity 属性，让物体在指定的方向产生移动速度。

```c#
public float speed = 5f;
public Rigidbody rigidbody;

void Start()
{
    rigidbody.velocity = Vector3.right * speed;
}
```

刚体的速度只需要设置一次，就会持续不断地移动。

（注：刚体默认是受到重力影响，会往下掉落。如果想要让物体不受重力影响，可以取消勾选刚体组件上的 Use Gravity 选项。）

当然，在实际项目中，这个速度需要不断地进行调整，不会只在 Start 设置一次。

对于物理组件来说，通常会使用 FixedUpdate 去控制物理逻辑，对于一些按键输入，则可以使用 Update 控制。

```c#
void Update()
{
    // 方向键左右的输入，取值为 -1, 0, 1
    float horizontal = Input.GetAxisRaw("Horizontal");
    rigidbody.velocity = Vector3.right * horizontal * speed;
}
```

现在物体就可以通过方向键进行左右移动了。