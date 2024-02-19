---
title: C# 基础语法 02 属性方法
date: 2024-01-07 16:07:34
tags: C#
---

简单复习一下 C# 基础语法（第二期）。

<!--more-->

# 类

类包含字段、属性和方法，是一个抽象概念。

对象是类的一个实例。

## 字段和属性

字段和属性定义了对象的特征。

声明一个学生类，包含若干个字段和属性。

```c#
public enum Gender
{
    Boy,
    Girl
}

public class Student
{
    // 字段
    public string name;    // 姓名（公开）
    public Gender gender;  // 性别（公开）
    private int age;       // 年龄（私有）

    // 属性
    public int Age
    {
        get // 可以获取年龄字段
        {
            return age;
        }
        set // 设置年龄字段时，可以添加逻辑判断，例如限制在 (0, 100) 之间
        {
            // value 是一个关键字，对属性进行赋值时，value 是等号右边的值
            if (value > 0 && value < 100)
            {
                age = value;
            }
        }
    }
}
```



实例化一个学生对象，对字段和属性进行赋值，并打印结果。

```c#
public class Program
{
    static void Main()
    {
        Student student = new Student();
        student.name = "小明";
        student.gender = Gender.Boy;
        student.Age = 200;
        
        Console.WriteLine(student.name);    // 小明
        Console.WriteLine(student.gender);  // Boy
        Console.WriteLine(student.Age);     // 0（对 Age 属性赋值时，数值不在规定范围内，age 字段默认值为 0）
    }
}
```



## 方法

方法定义了对象的行为，可以执行一系列的逻辑。

### 无参方法

没有参数的方法。

例如，声明一个 Say 方法，打印三个句子。

```c#
public class Student
{
    // ...

    public void Say()
    {
        Console.WriteLine("你好，我叫 " + name);
        Console.WriteLine("我是一个 " + gender);
        Console.WriteLine("我今年刚满 " + age + " 岁~~~");
    }
}
```



通过实例对象调用方法。

```c#
public class Program
{
    static void Main()
    {
        Student student = new Student();
        student.name = "喵喵球";
        student.gender = Gender.Girl;
        student.Age = 18;
        
        student.Say();
        
        // 你好，我叫 喵喵球
        // 我是一个 Girl
        // 我今年刚满 18 岁~~~
    }
}
```



### 有参方法

有参数的方法。

例如，声明一个 Repeat 方法，有一个 string 类型的参数，方法体内会复读传入的内容。

```c#
public class Student
{
    // ...

    public void Repeat(string content)
    {
        Console.WriteLine("我再说一遍 " + content);
    }
}
```



通过实例对象调用方法，并传入一个字符串参数。

```c#
public class Program
{
    static void Main()
    {
        Student student = new Student();

        student.Repeat("我今年刚满 18 岁~~~");
        
        // 我再说一遍 我今年刚满 18 岁~~~
    }
}
```



### 构造方法

构造方法的名称和类名相同。

当类里面没有写任何构造方法时，会有一个默认的无参构造方法。

```c#
public class Student
{
    // 无参构造方法，不写也行
    public Student()
    {

    }
}
```



如果需要在实例化对象时，有一些执行逻辑，就可以显式写出构造方法。

```c#
public class Student
{
    public Student()
    {
		Console.WriteLine("一个对象被实例化了");
    }
}
```



```c#
public class Program
{
    static void Main()
    {
        Student student = new Student();
        
        // 一个对象被实例化了
    }
}
```



也可以自定义一个有参构造方法。

当参数名和字段名相同时，为了区分是参数赋值给字段，就需要在使用字段时，加上 this。

```c#
public class Student
{
    // ...
    
    public Student(string name, Gender gender, int age)
    {
        this.name = name;
        this.gender = gender;
        this.age = age;
    }
}
```



此时，私有字段 age 可以被直接赋值。

```c#
public class Program
{
    static void Main()
    {
        Student student = new Student("小明", Gender.Boy, 10);
    }
}
```



### 析构方法

当对象被垃圾收集器回收时，会自动调用析构方法。

析构方法也跟类名相同，前面要加个 ~ 符号。

```c#
public class Student
{
    ~Student()
    {
        Console.WriteLine("我被回收了呜呜呜~~~");
    }
}
```



在 Program 类声明一个静态方法 CreateStudent，方法体内实例化一个对象，当方法调用结束后，对象就没有引用了。

此时可以调用 GC.Collect 方法主动回收一次垃圾，student 对象就会调用析构方法。

```c#
public class Program
{
    static void Main()
    {
        CreateStudent();
        GC.Collect();
        
        // 我被回收了呜呜呜~~~
    }
    
    static void CreateStudent()
    {
        Student student = new Student("小明", Gender.Boy, 10);
    }
}
```



### 静态方法

不用实例化对象就能调用的方法。

例如，上述的 CreateStudent 方法，前面有个修饰符 static，表示静态的，不需要 new 一个 Program 对象，就能够直接调用。



Student 类也可以声明一个静态方法。

```c#
public class Student
{
    public static void Introduce()
    {
        Console.WriteLine("我是一个学生");
    }
}
```



无需实例化对象，直接通过类名调用。

```c#
public class Program
{
    static void Main()
    {
        Student.Introduce();
        
        // 我是一个学生
    }
}
```

