---
title: C# 基础语法 03 面向对象
date: 2024-01-14 19:42:17
tags: C#
---

简单复习一下 C# 基础语法（第三期）。

<!--more-->

# 面向对象

## 封装

隐藏对象的信息，留出访问的接口，保护字段不被随意修改。

C# 的属性就是用来实现封装的。

例如，Hero 类有一个只读的等级属性，只能获取到英雄等级，而不能直接对等级进行修改。

如果要对英雄等级进行修改，只能通过给经验属性赋值，在类的内部进行经验和等级的转换。

```c#
public class Hero
{
    // 英雄等级 只读
    private int level;
    public int Level
    {
        get { return level; }
    }

    // 经验等级转换
    private int factor = 1000;
    public int Experience
    {
        get
        {
            return level * factor;
        }
        set
        {
            level = Math.Abs(value / factor);
        }
    }
}
```



创建英雄对象，给他增加经验值，查看他的等级。

```c#
public class Program
{
    static void Main()
    {
        Hero hero = new Hero();
        hero.Experience = 10000;
        Console.WriteLine("英雄等级：" + hero.Level);
        Console.WriteLine("英雄经验：" + hero.Experience);
        
        // 英雄等级：10
        // 英雄经验：10000
    }
}
```



## 继承

一个类继承另一个类，它就拥有了另一个类的所有字段、属性和方法。

继承的类叫子类（或派生类），被继承的类叫父类（或基类）。

继承可以减少代码重复，提升代码的复用性。



现在出现了两位英雄，他们都继承了 Hero 类，并且类的内部没有定义任何成员字段、属性和方法。

```c#
/// <summary>
/// 剑圣
/// </summary>
public class Blademaster : Hero
{

}

/// <summary>
/// 大魔法师
/// </summary>
public class Archmage : Hero
{

}
```



分别创建剑圣和大魔法师对象，给他们增加不同的经验值，查看他们的等级。

```c#
public class Program
{
    static void Main()
    {
        Blademaster blademaster = new Blademaster();
        blademaster.Experience = 10000;
        Console.WriteLine("剑圣等级：" + blademaster.Level);
        Console.WriteLine("剑圣经验：" + blademaster.Experience);

        Archmage archmage = new Archmage();
        archmage.Experience = 20000;
        Console.WriteLine("大魔法师等级：" + archmage.Level);
        Console.WriteLine("大魔法师经验：" + archmage.Experience);
        
        // 剑圣等级：10
        // 剑圣经验：10000
        // 大魔法师等级：20
        // 大魔法师经验：20000
    }
}
```



## 多态

多态是在继承的基础上，子类重写父类的方法，使用父类的类型声明变量，引用子类的实例对象，从而产生类型代差，调用同样的方法，却产生不一样的行为。

现在给英雄增加一个攻击的方法，这个方法需要使用 virtual 修饰符，是一个虚方法，表示可以被重写。

```c#
/// <summary>
/// 英雄
/// </summary>
public class Hero
{
    // ...
    
    public virtual void Attack()
    {
        Console.WriteLine("英雄攻击");
    }
}
```



让剑圣和大魔法师重写父类的攻击方法，重写方法需要使用 override 修饰符。

```c#
/// <summary>
/// 剑圣
/// </summary>
public class Blademaster : Hero
{
    public override void Attack()
    {
        Console.WriteLine("致命一击");
    }
}

/// <summary>
/// 大魔法师
/// </summary>
public class Archmage : Hero
{
    public override void Attack()
    {
        Console.WriteLine("暴风雪");
    }
}
```



现在，声明 Hero 类型的变量，引用不同类型的实例对象，调用 Attack。

```c#
public class Program
{
    static void Main()
    {
        // 普通英雄
        Hero hero = new Hero();
        hero.Attack();

        // 剑圣
        Hero blademaster = new Blademaster();
        blademaster.Attack();

        // 大魔法师
        Hero archmage = new Archmage();
        archmage.Attack();
        
        // 英雄攻击
        // 致命一击
        // 暴风雪
    }
}
```

这里之所以可以使用 Hero 变量引用 Blademaster 和 Archmage 对象，是因为他们继承了 Hero 类。

可以说剑圣是一个英雄，但反过来不行，英雄不一定是剑圣。

> 如果反过来写，Blademaster blademaster = new Hero(); 则会报错：无法将类型 Hero 隐式转换为 Blademaster。
>
> 如果要强制类型转换，运行起来也会报错，无法将 Hero 强制转换为 Blademaster：Unhandled exception. System.InvalidCastException: Unable to cast object of type 'Hero' to type 'Blademaster'.



同时，也可以使用数组统一管理。

```c#
public class Program
{
    static void Main()
    {
        Hero hero = new Hero();
        Hero blademaster = new Blademaster();
        Hero archmage = new Archmage();

        Hero[] heros = { hero, blademaster, archmage };
        for (int i = 0; i < heros.Length; i++)
        {
            heros[i].Attack();
        }
        
        // 英雄攻击
        // 致命一击
        // 暴风雪
    }
}
```



多态重写方法，实际上是在继承链上调用该方法的最新版本。

现在新增一个狂暴剑圣，继承了剑圣，也重写了 Attack 方法。

```c#
/// <summary>
/// 狂暴剑圣
/// </summary>
public class BerserkBlademaster : Blademaster
{
    public override void Attack()
    {
        Console.WriteLine("剑刃风暴");
    }
}
```



跟往常一样声明 Hero 类型变量，引用子类实例，调用方法。

```c#
public class Program
{
    static void Main()
    {
        Hero hero = new Hero();
        Hero blademaster = new Blademaster();
        Hero berserkBlademaster = new BerserkBlademaster();
        
        hero.Attack();
        blademaster.Attack();
        berserkBlademaster.Attack();
        
        // 英雄攻击
        // 致命一击
        // 剑刃风暴
    }
}
```



如果狂暴剑圣没有重写 Attack 方法，而是声明了一个相同名字的方法。

```c#
/// <summary>
/// 狂暴剑圣
/// </summary>
public class BerserkBlademaster : Blademaster
{
    public void Attack()
    {
        Console.WriteLine("剑刃风暴");
    }
}
```



那么狂暴剑圣对象被 Hero 类型变量引用时，调用 Attack 方法，此时该方法被重写后的最新版本是剑圣的 Attack 方法，不会输出剑刃风暴，而是致命一击。

除非用狂暴剑圣本来的类型，才会输出剑刃风暴。

```c#
public class Program
{
    static void Main()
    {
        Hero hero = new Hero();
        Hero blademaster = new Blademaster();
        Hero berserkBlademaster = new BerserkBlademaster();
        
        hero.Attack();
        blademaster.Attack();
        berserkBlademaster.Attack();
        
        // 英雄攻击
        // 致命一击
        // 致命一击
        
        BerserkBlademaster berserkBlademaster2 = new BerserkBlademaster();
        berserkBlademaster2.Attack();
        
        // 剑刃风暴
    }
}
```

