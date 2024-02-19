---
title: C# 基础语法 04 值和引用
date: 2024-01-21 21:03:57
tags: C#
---

简单复习一下 C# 基础语法（第四期）。

<!--more-->

# 值类型和引用类型

值类型，直接存储数据。

引用类型，存储内存地址，通过地址找到数据。



值类型：int，float，double，bool，char 等等。

引用类型：string，class 等等。



值类型赋值时，会创建值的副本。

修改值类型变量，不会影响其他值类型变量。

```c#
public class Program
{
    static void Main()
    {
        int x = 1;
        int y = x;
        x = 0;
        Console.WriteLine("x = " + x);
        Console.WriteLine("y = " + y);
        
        // x = 0
        // y = 1
    }
}
```



引用类型，两个变量引用的对象相同时，对其中一个变量引用的对象进行修改，另一个变量引用的对象也发生了变化。

因为这两个变量只是存储了一个地址，指向同一块数据区域。

就好比，两个人使用同一个银行账户，一个人取走了全部的钱，另一个人就没钱了。



例如，有一个账户类，包含余额字段，存钱、取钱、查询余额的方法。

```c#
public class Account
{
    private float money;

    public void Save(float money)
    {
        if (money > 0)
        {
            this.money += money;
            Console.WriteLine("存入 " + money);
        }
    }

    public void Draw(float money)
    {
        if (money > 0)
        {
            if (this.money >= money)
            {
                this.money -= money;
                Console.WriteLine("取出 " + money);
            }
            else
            {
                Console.WriteLine("余额不足");
            }
        }
    }

    public void Query()
    {
        Console.WriteLine("余额 " + money);
    }
}
```



现在创建一个账户对象，赋值给 myAccount，接着再赋值给 herAccount，两个 Account 变量引用的是同一个对象。

在我的账户存入 10 元，从她的账户取出 10 元，我的账户余额 0 元。

```c#
public class Program
{
    static void Main()
    {
        Account myAccount = new Account();    // 我的账户
        Account herAccount = myAccount;       // 她的账户

        myAccount.Save(10f);    // 我的账户存入 10 元
        herAccount.Draw(10f);   // 她的账户取出 10 元
        myAccount.Query();      // 我的账户余额  0 元
        
        // 存入 10
        // 取出 10
        // 余额 0
    }
}
```



# 装箱和拆箱

装箱，把值类型转换成引用类型。

拆箱，把引用类型转换成值类型。

```c#
public class Program
{
    static void Main()
    {
        int i = 1;
        
        object obj = i;     // 装箱
        
        int j = (int)obj;   // 拆箱
    }
}
```

有时候为了通用，会把方法的参数类型定义为 object，当传入一个值类型参数时，就会需要装箱。

由于装箱会生成新的对象，对运行效率有一定影响，一般是尽量避免装箱和拆箱的。

装箱和拆箱时，对两个变量的值进行修改，是互不影响的。



# ref 和 out

通常，当方法的参数是值类型时，外部传入的变量和方法体内的参数变量，是分别独立的，对方法体内的参数变量修改时，不会影响到外部的变量。

这是因为方法内的形参，只是外部传入的实参的一个副本。

```c#
public class Program
{
    static void Main()
    {
        int i = 0;
        Add(i);
        Console.WriteLine("i = " + i);
        
        // i = 0
    }

    static void Add(int i)
    {
        i++;
    }
}
```



如果要实现当 i 传入 Add 方法内，执行之后 i 的数值增加，则需要使用 ref 关键字。

方法的参数类型前面要加个 ref，调用方法时，传入的参数前面也要加个 ref。

此时，i 变成了按引用传参，在方法内对 i 进行修改，同时也会影响到外部的 i。

```c#
public class Program
{
    static void Main()
    {
        int i = 0;
        Add(ref i);
        Console.WriteLine("i = " + i);
        
        // i = 1
    }

    static void Add(ref int i)
    {
        i++;
    }
}
```



需要注意的是，外部的 i 需要赋初始值，如果没有赋值，则会报错。

ref 的特点是有进有出，需要先赋值，再传入，有输出。



out 也是按引用传参，不过 out 只是用来输出多个参数。

out 的特点是只出不进，不需要先赋值，方法会把内部的数值输出到外部的变量，即使赋初始值，也会被输出的数值覆盖。

```c#
public class Program
{
    static void Main()
    {
        int i;
        int j = 3;
        Add(out i, out j);
        Console.WriteLine("i = " + i);
        Console.WriteLine("j = " + j);
        
        // i = 1
        // j = 2
    }

    static void Add(out int i, out int j)
    {
        i = 1;
        j = 2;
    }
}
```

