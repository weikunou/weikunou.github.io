---
title: C# 基础语法 06 委托事件
date: 2024-02-04 20:47:50
tags: C#
---

简单复习一下 C# 基础语法（第六期）。

<!--more-->

# 委托

委托是一个类，可以被实例化，包装一系列的方法，相当于一个方法的容器，可以作为参数传入其他方法。

### 模板

方法的参数列表内，有一个委托参数，方法体会调用这个委托参数。

而委托参数包装的是什么样的方法，则由外部传入的方法决定。

相当于一个填空题，借助外部的方法产生结果。

模板委托方法通常在代码的中间部分，有返回值。



定义一个 Person 类，包含人物的名字和正在思考的内容。

同时，定义一个 Answer 类，用来包装人物答题的内容。

```c#
public class Person
{
    public string person;
    public string result;

    public Person(string name, string thinking)
    {
        person = name;
        result = thinking;
    }

    public Answer Think()
    {
        Answer answer = new Answer();
        answer.person = person;
        answer.result = result;
        return answer;
    }
}

public class Answer
{
    public string person;
    public string result;
}
```



再定义一个 Exam 类，输出答题人和答案。

其中，方法的参数是一个委托，Func 类是 C# 内部定义好的一个委托类，它可以有返回值，尖括号 < > 内填入返回值的类型。

在方法内部调用委托，得到 Answer 类型的返回值，把内容打印输出。

```c#
public class Exam
{
    public void AnswerQuestion(Func<Answer> solution)
    {
        Answer answer = solution.Invoke();
        Console.WriteLine($"答题人：{answer.person} 答案：{answer.result}");
    }
}
```



实际使用时，可以定义 Func 变量，new 一个委托实例，将一个实例的方法作为参数传入。

然后把委托变量传给 AnswerQuestion 方法，同一个方法，可以接收不同的委托变量，实现功能的复用。

```c#
public class Program
{
    static void Main()
    {
        Exam exam = new Exam();
        Person person1 = new Person("小明", "鸡排");
        Person person2 = new Person("小红", "奶茶");

        Func<Answer> func1 = new Func<Answer>(person1.Think);
        Func<Answer> func2 = new Func<Answer>(person2.Think);

        exam.AnswerQuestion(func1);
        exam.AnswerQuestion(func2);
        
        // 答题人：小明 答案：鸡排
        // 答题人：小红 答案：奶茶
    }
}
```



### 回调

同样，方法的参数列表内，有一个委托参数。

在执行完一系列代码之后，调用外部方法进行回调。

回调委托方法通常在代码的末尾部分，无返回值。



继续上面的例子，添加一个 Logger 类，用于答题后的信息提示。

```c#
public class Logger
{
    public string message;

    public Logger(string message)
    {
        this.message = message;
    }

    public void Log() 
    {
        Console.WriteLine(message);
    }
}
```



扩展 AnswerQuestion 的参数列表，并在打印答题结果之后进行回调。

Action 类也是 C# 内部定义好的一个委托类，无返回值。

```c#
public class Exam
{
    public void AnswerQuestion(Func<Answer> solution, Action callback)
    {
        Answer answer = solution.Invoke();
        Console.WriteLine($"答题人：{answer.person} 答案：{answer.result}");
        callback.Invoke();
    }
}
```



实例化 Logger 类，填入信息提示。

可以定义 Action 变量并实例化，传入 Logger 类的 Log 方法。

然后把 action 变量作为参数，作为回调方法传入 AnswerQuestion 方法中。

现在每次打印答题结果之后，都会有答题成功的信息提示。

```c#
public class Program
{
    static void Main()
    {
        Exam exam = new Exam();
        Person person1 = new Person("小明", "鸡排");
        Person person2 = new Person("小红", "奶茶");

        Func<Answer> func1 = new Func<Answer>(person1.Think);
        Func<Answer> func2 = new Func<Answer>(person2.Think);

        Logger logger = new Logger("答题成功");
        Action action = new Action(logger.Log);

        exam.AnswerQuestion(func1, action);
        exam.AnswerQuestion(func2, action);
        
        // 答题人：小明 答案：鸡排
        // 答题成功
        // 答题人：小红 答案：奶茶
        // 答题成功
    }
}
```



### 多播

委托是一对多的，也就是说，可以包装多个方法，当委托被调用时，会有多个方法被同时调用，调用的顺序则是方法被填入委托变量时的顺序。

在上述的例子进行扩展，再实例化一个 Logger，会提示【请继续答题】，并且使用 += 符号给 action 变量再添加一个方法引用。

此时回调方法会输出两行。

```c#
public class Program
{
    static void Main()
    {
        // ...

        Logger logger1 = new Logger("答题成功");
        Logger logger2 = new Logger("请继续答题");
        Action action = new Action(logger1.Log);
        action += logger2.Log;

        exam.AnswerQuestion(func1, action);
        exam.AnswerQuestion(func2, action);
        
        // 答题人：小明 答案：鸡排
        // 答题成功
        // 请继续答题
        // 答题人：小红 答案：奶茶
        // 答题成功
        // 请继续答题
    }
}
```



# 事件

事件是在委托的基础上，对委托的访问进行限制，事件的右侧只能是 += 或 -= 符号，不能被 = 符号直接覆盖，也不能被 .Invoke 调用。

接着上述的例子，自定义一个 SubmitEventHandler 委托，和一个 SubmitSystem 类。

SubmitSystem 类包含一个委托字段，和一个事件。

事件必须同时包含 add 和 remove，对委托字段进行添加方法和移除方法。

同时，因为委托字段是私有的，外部无法调用，事件也不能被调用，所以提供一个方法 TriggerSubmit 对委托字段进行调用。

> 注意：如果 submitEventHandler 没有包含任何方法，则会是 null，需要进行判空。



```c#
public delegate void SubmitEventHandler();

public class SubmitSystem
{
    private SubmitEventHandler submitEventHandler;

    public event SubmitEventHandler OnSubmit
    {
        add
        {
            submitEventHandler += value;
        }
        remove
        {
            submitEventHandler -= value;
        }
    }

    public void TriggerSubmit()
    {
        if (submitEventHandler != null)
        {
            submitEventHandler.Invoke();
        }
    }
}
```



现在，继续添加主程序的逻辑。

实例化 submitSystem 变量，给它的事件添加两个方法，并通过 TriggerSubmit 调用被事件保护起来的私有委托。

```c#
public class Program
{
    static void Main()
    {
        // ...
        
        Logger logger3 = new Logger("小明提交了试卷");
        Logger logger4 = new Logger("小红提交了试卷");
        
        SubmitSystem submitSystem = new SubmitSystem();
        submitSystem.OnSubmit += logger3.Log;
        submitSystem.OnSubmit += logger4.Log;
        
        submitSystem.TriggerSubmit();
        
        // ...
        // 小明提交了试卷
        // 小红提交了试卷
    }
}
```



上述事件的定义是一个完整定义，实际上可以简化。

可以直接定义事件，无需定义私有委托字段，也无需为事件添加 add 和 remove 访问器。

```c#
public delegate void SubmitEventHandler();

public class SubmitSystem
{
    public event SubmitEventHandler OnSubmit;

    public void TriggerSubmit()
    {
        if (OnSubmit != null)
        {
            OnSubmit.Invoke();
        }
    }
}
```



此时，事件的右侧似乎可以被 .Invoke 调用，还可以判空。

实际上，因为 C# 会在编译时自动生成私有委托字段，但是编写代码时并没有私有委托字段，所以不得不使用事件来触发调用，不过外部对事件的访问依然只能是 += 和 -= 符号。

