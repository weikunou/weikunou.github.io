---
title: C# 基础语法 07 抽象接口
date: 2024-02-12 12:32:52
tags: C#
---

简单复习一下 C# 基础语法（第七期 完结）。

<!--more-->

# 抽象类

抽象类是未完全实现逻辑的类，专门作为基类来使用，将具体逻辑推迟到合适的派生类去实现。



下面是一个具体类 FlyObject，包含一个虚方法 Fly，它可以有方法体，此时方法体内是空的，没有需要实现的逻辑。

由它衍生出两个具体类，Bird 和 Plane，它们都重写了 Fly 方法，并且加上了具体逻辑。

```c#
public class FlyObject
{
    public virtual void Fly()
    {
        
    }
}

public class Bird : FlyObject
{
    public override void Fly()
    {
        Console.WriteLine("小鸟在飞行");
    }
}

public class Plane : FlyObject
{
    public override void Fly()
    {
        Console.WriteLine("飞机在飞行");
    }
}
```



使用时

```c#
public class Program
{
    static void Main()
    {
        FlyObject bird = new Bird();
        bird.Fly();
        // 小鸟在飞行

        FlyObject plane = new Plane();
        plane.Fly();
        // 飞机在飞行
    }
}
```



由于基类 FlyObject 的 Fly 方法并没有具体实现逻辑，只是一个空的方法，此时可以将基类的 Fly 方法改成一个抽象方法，并去掉方法体。

因为包含了抽象方法，FlyObject 也需要加上 abstract 变成一个抽象类。

```c#
public abstract class FlyObject
{
    public abstract void Fly();
}
```



抽象类不能被实例化，需要有派生类继承它，通过派生类实例化。

继承了抽象类的派生类，必须通过重写，实现抽象方法。



# 接口

接口是完全未实现逻辑的“类”，只有方法成员。

和抽象类一样，接口也不能被实例化，需要有其他类去实现接口。



```c#
public interface IFlyObject
{
    void Fly();
}

public class Bird : IFlyObject
{
    public void Fly()
    {
        Console.WriteLine("小鸟在飞行");
    }
}

public class Plane : IFlyObject
{
    public void Fly()
    {
        Console.WriteLine("飞机在飞行");
    }
}
```



接口约定以 I 为开头进行命名，接口内定义的方法成员必须是 public 的，可以省略不写。

实现接口的类，必须包含接口内定义的所有方法成员，并且不需要写 override。



使用时

```c#
public class Program
{
    static void Main()
    {
        IFlyObject bird = new Bird();
        bird.Fly();

        IFlyObject plane = new Plane();
        plane.Fly();
    }
}
```



和抽象类不同的是，抽象类只能继承一个，接口可以同时实现多个。

