---
title: C# 基础语法 01
date: 2024-01-03 22:51:52
tags: C#
---

简单复习一下 C# 基础语法。

<!--more-->

# 基础语法

## 注释

```c#
// 单行注释

/*
 * 多行注释
 */

/// <summary>
/// 文档注释，会显示在代码提示框里
/// <summary>
```



## 输入输出

```c#
// 打印输出
Console.Write("输出");
Console.WriteLine("输出并换行");

// 读取输入
Console.ReadLine();
```



## 关键字

在 C# 中被规定了用途的单词，声明变量时，变量名不可与关键字冲突。

```c#
namespace // 命名空间
using     // 引入命名空间
class     // 声明类
// ...
```



## 数据类型

```c#
char gender = '男';      // 字符型
string name = "柯南";    // 字符串型
int age = 3;            // 整型
float money = 0.1f;     // 单精度浮点型
double salary = 0.1;    // 双精度浮点型
bool hasMoney = false;  // 布尔型
```



## 常量

声明并赋值，不可修改。

```c#
const double PI = 3.14;
```



## 变量

声明时可以赋值，如果没有赋值则会有相应类型的默认值。

```c#
double money = 0.1;   // 初始赋值
int num;              // 默认值 0
```



## 类型转换

低精度类型会自动转换成高精度类型

高精度类型需要强制转换成低精度类型，并且会丢失部分精度。

```c#
double i = 2;     // 自动转换（隐式转换），int 转 double
int j = (int)3.1; // 强制转换（显式转换），double 转 int，3.1 被转换成 3 再赋值给 j，丢失部分精度（小数点后的数字）
```



## 枚举

枚举值是从 0 递增的整数。

使用枚举可以限制变量只能从有限的选项中取值，避免随意取值。

```c#
// 声明时
enum Gender // 性别只能是男、女
{
    Boy,
    Girl
}

// 使用时
Gender.Boy
Gender.Girl
```



## 数组

### 一维数组

```c#
string[] students = new string[3]; // 数组长度为 3

// 三个等效语句
string[] students1 = new string[] { "A", "B", "C" };  // 直接赋值
string[] students2 = { "A", "B", "C" };               // 直接赋值
string[] students3 = new string[3] { "A", "B", "C" }; // 数组长度为 3，并赋值

Console.WriteLine(students1[0]);  // 输出 A
```



### 二维数组

```c#
int[,] scores = new int[3, 2] { { 11, 23 }, { 25, 44 }, { 76, 13 } };  // 3 行 2 列
// 11 23
// 25 44
// 76 13
Console.WriteLine(scores[0, 0]); // 取第一行第一列，结果 11
Console.WriteLine(scores[1, 1]); // 取第二行第二列，结果 44
```



## 条件分支

### if 语句

```c#
bool hasMoney = false;
bool hasEnergy = false;

if (hasMoney)
{
    Console.WriteLine("买买买");
}
else if (hasEnergy)
{
    Console.WriteLine("冲冲冲");
}
else
{
    Console.WriteLine("发呆");
}

// 输出
// 发呆
```



### switch 语句

```c#
string name = "小樱";

switch (name)
{
    case "小樱":
        Console.WriteLine("魔法杖");
        break;
    case "知世":
        Console.WriteLine("摄像机");
        break;
    default:
        Console.WriteLine("未知");
        break;
}

// 输出
// 魔法杖
```



## 循环

### for 循环

```c#
for (int i = 0; i < 5; i++)
{
    if (i == 3)
    {
        continue; // 跳过当次循环
    }
    // 打印 i
    Console.WriteLine(i);
}

// 输出
// 0
// 1
// 2
// 4
```



### foreach 循环

```c#
int[] num = {1, 2, 3};

foreach (int x in num)
{
    // 打印 x
    Console.WriteLine(x);
    break; // 跳出循环
}

// 输出
// 1
```



### while 循环

```c#
int i = 0;

while (i < 3)
{
    // 打印 i
    Console.WriteLine(i);
    i++;
}

// 输出
// 0
// 1
// 2
```



### do while 循环

```c#
int i = 1;

// 至少执行一次
do
{
    // 打印 i
    Console.WriteLine(i);
    i++;
} while (i < 0);

// 输出
// 1
```

