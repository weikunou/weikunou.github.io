---
title: Unity3D MD5 签名算法
date: 2024-08-11 11:22:47
tags: Unity3D
---

Unity3D MD5 签名算法实现流程。

<!--more-->

# MD5 签名算法

之前在和运营对接支付订单模块时，需要向他们的服务器发送查单请求，其中有个参数是 sign，要求把订单的相关信息，转换成 MD5 的形式。

简要流程：

1. 先对参数列表按照 ASCII 码排序
2. 参数列表拼接成字符串，末尾拼接密钥
3. 拼接后的字符串做 MD5 计算

## 参数列表排序

首先，假设有如下参数：

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestMD5 : MonoBehaviour
{
    void Start()
    {
        Dictionary<string, object> args = new Dictionary<string, object>
        {
            ["product_id"] = 100001,         // 产品 id
            ["product_name"] = "金币礼包",    // 产品名称
            ["amount"] = 1,                  // 数量
            ["price"] = 600                  // 价格
        };
    }
}
```

现在需要先对字典中的参数按照 ASCII 码进行排序。

这里创建一个静态类 `MD5Sign.cs` 作为工具类使用。

添加方法 `AsciiDicToStr`，接收字典类型的参数，返回字符串。

引用 `System.Linq` 命名空间，可以扩展字典的方法，例如 ToArray，把字典的 key 值取出，变成数组。

利用 `Array.Sort` 进行排序（需要命名空间 `System`），传入参数 `string.CompareOrdinal` 就可以按照 ASCII 码从小到大进行排序了。

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

public static class MD5Sign
{
    /// <summary>
    /// 以参数名的 ASCII 码从小到大排序并拼接
    /// </summary>
    /// <param name="args">参数列表</param>
    static string AsciiDicToStr(Dictionary<string, object> args)
    {
        // 取出 key 集合并转换成数组
        string[] keysArray = args.Keys.ToArray();

        // 对 key 数组进行排序（ASCII 码从小到大）
        Array.Sort(keysArray, string.CompareOrdinal);

        return "";
    }
}
```



## 参数拼接字符串

继续扩展 `AsciiDicToStr` 方法。

引用命名空间 `System.Text`，就可以使用 `StringBuilder` 来拼接字符串了。

遍历 key 数组，按照 `key=value&` 的形式拼接，参数值为 null 或者 "" 不参与签名。

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Text;

public static class MD5Sign
{
    /// <summary>
    /// 以参数名的 ASCII 码从小到大排序并拼接
    /// </summary>
    /// <param name="args">参数列表</param>
    static string AsciiDicToStr(Dictionary<string, object> args)
    {
        // 取出 key 集合并转换成数组
        string[] keysArray = args.Keys.ToArray();

        // 对 key 数组进行排序（ASCII 码从小到大）
        Array.Sort(keysArray, string.CompareOrdinal);

        // 按照排序后的 key 数组依次从字典中取出值进行拼接
        StringBuilder sb = new StringBuilder();

        foreach (var key in keysArray)
        {
            string value = args[key]?.ToString();

            // 参数值为 null 或者 "" 不参与签名
            if (value != null && !string.Empty.Equals(value))
            {
                sb.Append(key);
                sb.Append("=");
                sb.Append(value);
                sb.Append("&");
            }
        }

        return sb.ToString();
    }
}
```

现在可以再添加一个方法 `ComputeSign` 提供给外部调用。

这里定义了一个密钥，内容是随便输入的，用于演示，通常情况下密钥不会放在客户端，防止被破解获取，应该放到服务端。

调用上面写的 `AsciiDicToStr` 方法，传入字典，得到 result，在末尾继续拼接 `key=signKey`。

```csharp
public static class MD5Sign
{
    // 密钥，通常应该放到服务端，此处为了演示先放客户端
    static string signKey = "abcdefg";

    /// <summary>
    /// 计算参数签名
    /// </summary>
    /// <param name="args">参数列表</param>
    public static string ComputeSign(Dictionary<string, object> args)
    {
        // 按照 ASCII 码从小到大排序并拼接
        string result = AsciiDicToStr(args);

        // 拼接密钥
        result += $"key={signKey}";

        return result;
    }

    // ...
}
```

现在可以先调用这个方法，查看拼接后的结果。

```csharp
public class TestMD5 : MonoBehaviour
{
    void Start()
    {
        Dictionary<string, object> args = new Dictionary<string, object>
        {
            ["product_id"] = 100001,         // 产品 id
            ["product_name"] = "金币礼包",    // 产品名称
            ["amount"] = 1,                  // 数量
            ["price"] = 600                  // 价格
        };

        string result = MD5Sign.ComputeSign(args);
        Debug.Log($"result = {result}");
        
        // result = amount=1&price=600&product_id=100001&product_name=金币礼包&key=abcdefg
    }
}
```



## 计算 MD5

在静态类 `MD5Sign.cs` 中继续添加方法 `ComputeStringMD5`，接收字符串，返回字符串。

在 C# 中，有提供用于加密算法的类，需要引用命名空间 `System.Security.Cryptography`。

创建一个 MD5 对象，先通过 `Encoding.UTF8.GetBytes` 方法，把字符串转成 byte 数组，传给 MD5 对象的 `ComputeHash` 方法，同样得到 byte 数组，遍历这个 byte 数组，使用 `ToString("X2")` 的格式进行拼接（`X2` 表示使用大写十六进制，不足两位时前面补 0）。

因为 byte 数组的长度是 16，每个值都转成两位字符串，最终得到的就是 32 位字符串。

```csharp
// ...
using System.Security.Cryptography;

public static class MD5Sign
{
    // ...

    /// <summary>
    /// 计算字符串的 MD5 值
    /// </summary>
    /// <param name="str">字符串</param>
    static string ComputeStringMD5(string str)
    {
        StringBuilder sb = new StringBuilder();

        // 创建 MD5 对象
        using (MD5 md5 = MD5.Create())
        {
            // 字符串转 byte 数组
            byte[] buffer = Encoding.UTF8.GetBytes(str);

            // 计算 MD5 值
            byte[] mdBytes = md5.ComputeHash(buffer);
            md5.Clear();

            // 将 byte 数组中的每个元素以大写十六进制字符串的方式拼接
            for (int i = 0; i < mdBytes.Length; i++)
            {
                sb.Append(mdBytes[i].ToString("X2"));
            }

            return sb.ToString();
        }
    }
}
```

最后把这个方法接入 `ComputeSign` 方法中。

```csharp
public static class MD5Sign
{
    /// <summary>
    /// 计算参数签名
    /// </summary>
    /// <param name="args">参数列表</param>
    public static string ComputeSign(Dictionary<string, object> args)
    {
        // 按照 ASCII 码从小到大排序并拼接
        string result = AsciiDicToStr(args);

        // 拼接密钥
        result += $"key={signKey}";

        // 计算拼接后的字符串的 MD5 值
        string md5 = ComputeStringMD5(result);

        return md5;
    }
}
```

现在调用这个方法就可以得到计算 MD5 之后的结果了。

```csharp
public class TestMD5 : MonoBehaviour
{
    void Start()
    {
        Dictionary<string, object> args = new Dictionary<string, object>
        {
            ["product_id"] = 100001,         // 产品 id
            ["product_name"] = "金币礼包",    // 产品名称
            ["amount"] = 1,                  // 数量
            ["price"] = 600                  // 价格
        };

        string result = MD5Sign.ComputeSign(args);
        Debug.Log($"result = {result}");
        
        // result = EF4834EF466F73319545057E703E73E2
    }
}
```

# 完整代码

`MD5Sign.cs`

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Security.Cryptography;

public static class MD5Sign
{
    // 密钥，通常应该放到服务端，此处为了演示先放客户端
    static string signKey = "abcdefg";

    /// <summary>
    /// 计算参数签名
    /// </summary>
    /// <param name="args">参数列表</param>
    public static string ComputeSign(Dictionary<string, object> args)
    {
        // 按照 ASCII 码从小到大排序并拼接
        string result = AsciiDicToStr(args);

        // 拼接密钥
        result += $"key={signKey}";

        // 计算拼接后的字符串的 MD5 值
        string md5 = ComputeStringMD5(result);

        return md5;
    }

    /// <summary>
    /// 以参数名的 ASCII 码从小到大排序并拼接
    /// </summary>
    /// <param name="args">参数列表</param>
    static string AsciiDicToStr(Dictionary<string, object> args)
    {
        // 取出 key 集合并转换成数组
        string[] keysArray = args.Keys.ToArray();

        // 对 key 数组进行排序（ASCII 码从小到大）
        Array.Sort(keysArray, string.CompareOrdinal);

        // 按照排序后的 key 数组依次从字典中取出值进行拼接
        StringBuilder sb = new StringBuilder();

        foreach (var key in keysArray)
        {
            string value = args[key]?.ToString();

            // 参数值为 null 或者 "" 不参与签名
            if (value != null && !string.Empty.Equals(value))
            {
                sb.Append(key);
                sb.Append("=");
                sb.Append(value);
                sb.Append("&");
            }
        }

        return sb.ToString();
    }

    /// <summary>
    /// 计算字符串的 MD5 值
    /// </summary>
    /// <param name="str">字符串</param>
    static string ComputeStringMD5(string str)
    {
        StringBuilder sb = new StringBuilder();

        // 创建 MD5 对象
        using (MD5 md5 = MD5.Create())
        {
            // 字符串转 byte 数组
            byte[] buffer = Encoding.UTF8.GetBytes(str);

            // 计算 MD5 值
            byte[] mdBytes = md5.ComputeHash(buffer);
            md5.Clear();

            // 将 byte 数组中的每个元素以大写十六进制字符串的方式拼接
            for (int i = 0; i < mdBytes.Length; i++)
            {
                sb.Append(mdBytes[i].ToString("X2"));
            }

            return sb.ToString();
        }
    }
}
```
