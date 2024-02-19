---
title: C# 基础语法 05 泛型集合
date: 2024-01-28 14:31:45
tags: C#
---

简单复习一下 C# 基础语法（第五期）。

<!--more-->

# 泛型集合

## 泛型

泛型是一个类型占位符，在定义泛型类的时候，可以先使用一个字母 T 占位，在实际使用时，需要传入一个具体的类型替代 T。

例如，定义一个泛型类，后面的尖括号 < > 先填 T，并且使用 T 声明一个变量 myType，在构造函数给 myType 赋值。

```c#
public class MyGeneric<T>
{
    public T myType;

    public MyGeneric(T type)
    {
        myType = type;
    }
}
```



使用时，需要填入具体的类型替代 T。

通过 myType 字段的 GetType 方法，查看它的数据类型。

填入 int 的泛型实例，它的 myType 字段的类型是 System.Int32。

填入 string 的泛型实例，它的 myType 字段的类型是 System.String。

```c#
public class Program
{
    static void Main()
    {
        MyGeneric<int> myGenericInt = new MyGeneric<int>(1);
        Console.WriteLine(myGenericInt.myType.GetType());
        // System.Int32

        MyGeneric<string> myGenericString = new MyGeneric<string>("小明");
        Console.WriteLine(myGenericString.myType.GetType());
        // System.String
    }
}
```



这样，泛型类就可以通过填入不同的数据类型，提升代码的复用性，无需将类型写死。



## 列表（List）

列表是数组的扩展，内部实现是以数组为基础的。

列表的使用方法主要有：

1. 添加元素（Add）
2. 访问元素（[]）
3. 删除元素（Remove）
4. 清空列表（Clear）



使用 List 声明一个列表，尖括号 < > 内填入数据类型。

使用 Add 添加三个元素，并用 for 循环访问每个元素，打印输出。其中，Count 可以获取列表的长度。

```c#
public class Program
{
    static void Main()
    {
        List<int> nums = new List<int>();
        nums.Add(1);
        nums.Add(2);
        nums.Add(3);
        for (int i = 0; i < nums.Count; i++)
        {
            Console.WriteLine(nums[i]);
        }
        
        // 1
        // 2
        // 3
    }
}
```



使用 Remove 删除一个元素，后续的元素都会往前移动，列表长度减少。

```c#
public class Program
{
    static void Main()
    {
        List<int> nums = new List<int>();
        nums.Add(1);
        nums.Add(2);
        nums.Add(3);
        nums.Remove(2);

        for (int i = 0; i < nums.Count; i++)
        {
            Console.WriteLine(nums[i]);
        }
        
        // 1
        // 3
    }
}
```



使用 Clear 清空列表。

打印列表的长度，输出 0，表示列表中没有任何元素了。

```c#
public class Program
{
    static void Main()
    {
        List<int> nums = new List<int>();
        nums.Add(1);
        nums.Add(2);
        nums.Add(3);
        Console.WriteLine(nums.Count);
        // 3
        
        nums.Clear();
        Console.WriteLine(nums.Count);
        // 0
    }
}
```



## 字典（Dictionary）

字典存储一系列键值对，通过 key 值，可以快速获取到对应的 value 值。

和列表类似，字典的使用方法主要有：

1. 添加元素（Add）
2. 访问元素（[]）
3. 删除元素（Remove）
4. 判断是否包含某个 key 值（ContainsKey）
5. 清空列表（Clear）



使用 Dictionary 声明一个字典，尖括号 < > 内填入两个数据类型，第一个是 key，第二个是 value。

使用 Add 添加三个元素，记录每个学生的分数。

通过 [] 访问字典元素，填入 key 值获取 value 值。

```c#
public class Program
{
    static void Main()
    {
        Dictionary<string, int> scores = new Dictionary<string, int>();
        scores.Add("小明", 60);
        scores.Add("小红", 80);
        scores.Add("小坤", 100);

        Console.WriteLine("小明的分数：" + scores["小明"]);
        Console.WriteLine("小红的分数：" + scores["小红"]);
        Console.WriteLine("小坤的分数：" + scores["小坤"]);
        
        // 小明的分数：60
        // 小红的分数：80
        // 小坤的分数：100
    }
}
```



使用 Remove 删除某个 key 对应的元素之后，再次使用此 key 值去访问字典，则会报错。

所以通常要判断字典是否包含某个 key 值，才能去访问。

例如，从 scores 字典中删除小红的数据，如果直接访问 scores["小红"] 会报错，需要使用 ContainsKey 判断一下是否包含小红的数据。

```c#
public class Program
{
    static void Main()
    {
        Dictionary<string, int> scores = new Dictionary<string, int>();
        scores.Add("小明", 60);
        scores.Add("小红", 80);
        scores.Add("小坤", 100);

        Console.WriteLine("小明的分数：" + scores["小明"]);
        Console.WriteLine("小坤的分数：" + scores["小坤"]);

        scores.Remove("小红");

        if (scores.ContainsKey("小红"))
        {
            Console.WriteLine("小红的分数：" + scores["小红"]);
        }
    }
}
```



使用 Clear 清空字典。

字典同样有 Count 属性，获取字典的键值对的个数。

```c#
public class Program
{
    static void Main()
    {
        Dictionary<string, int> scores = new Dictionary<string, int>();
        scores.Add("小明", 60);
        scores.Add("小红", 80);
        scores.Add("小坤", 100);
        Console.WriteLine(scores.Count);
        // 3
        
        scores.Clear();
        Console.WriteLine(scores.Count);
        // 0
    }
}
```

