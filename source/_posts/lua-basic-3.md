---
title: Lua 基础 03 常用函数
date: 2024-05-26 12:52:34
tags: Lua
---

Lua 基础相关知识 第三期

<!--more-->

# 字符串

## 格式化字符串 `string.format`

通常字符串的连接可以使用 .. 符号，不过当字符串比较长，这样的连接方式就很繁琐，这时可以使用 `string.format` 进行格式化。

常用的格式控制符：

- %s 接收一个字符串
- %d 接收一个数字并转化为有符号整数，%02d 可以显示两位数，个位数时前面补 0，通常用于倒计时格式
- %f 接收一个数字并转化为浮点数，默认保留 6 位小数，后面补 0，%.2f 可以保留两位小数

```lua
local str = "hello"
local result = string.format("str = %s", str)
print(result)

-- str = hello

local minute = 2
local second = 30
local result = string.format("%02d:%02d", minute, second)
print(result)

-- 02:30

local multi = 0.214
local result = string.format("%.2f", multi)
print(result)

-- 0.21
```

## 字符串长度 `string.len`

通常只包含英文的字符串，可以使用 `string.len` 计算字符串长度。

```lua
local str = "hello"
print(string.len(str))  -- 5
```

如果字符串包含中文，那么就需要使用 `utf8.len` 计算 ASCII 字符。

可以看到，`string.len` 把小明的长度计算为 6，`utf8.len` 则是 2。空格的长度为 1。

```lua
local str = "hello 小明"
print(string.len(str))  -- 12
print(utf8.len(str))    -- 8
```

## 字符串查找 `string.find`

在一个字符串中查找指定的子串内容，如果找到第一个匹配的子串（后续的不查找），则返回这个子串的开始位置和结束位置，否则返回 nil。

```lua
local str = "hello lua"
local startPos, endPos = string.find(str, "lua")
print(startPos, endPos)

-- 7       9
```

如果要倒着查找，可以在第三个参数位置指定倒数多少个字符。

例如，倒数一个字符，因为要查找的 lua 是三个字符，只查找一个字符肯定是不匹配的。如果倒数三个字符，就正好能匹配。

```lua
local str = "hello lua"
local startPos, endPos = string.find(str, "lua", -1)
print(startPos, endPos)

-- nil     nil

local str = "hello lua"
local startPos, endPos = string.find(str, "lua", -3)
print(startPos, endPos)

-- 7       9
```

## 字符串截取 `string.sub`

指定截取的开始位置和结束位置，如果是负数，则从倒数位置开始截取。

```lua
local str = "hello lua"
local strSub = string.sub(str, 1, 3)
print(strSub)

-- hel

local str = "hello lua"
local strSub = string.sub(str, -3)
print(strSub)

-- lua
```

## 字符串替换 `string.gsub`

指定被替换和要替换的字符，还可以指定替换次数，如果没有指定替换次数，默认全部替换。

```lua
local str = "hello lua"
local strSub = string.gsub(str, "l", "a", 2)
print(strSub)

-- heaao lua
```

# 表

## 插入 `table.insert`

在表的末尾插入新的元素，也可以指定插入的位置。

```lua
local names = { "Alice", "Bob" }
table.insert(names, "Jack")  -- 在末尾插入
for key, value in pairs(names) do
    print(key, value)
end

-- 1       Alice
-- 2       Bob
-- 3       Jack

local names = { "Alice", "Bob" }
table.insert(names, 1, "Jack")  -- 在索引为 1 的位置插入（也可以认为是在开头插入）
for key, value in pairs(names) do
    print(key, value)
end

-- 1       Jack
-- 2       Alice
-- 3       Bob
```

## 移除 `table.remove`

从表的末尾移除一个元素，也可以指定移除的位置。

```lua
local names = { "Alice", "Bob", "Jack" }
table.remove(names)  -- 移除末尾
for key, value in pairs(names) do
    print(key, value)
end

-- 1       Alice
-- 2       Bob

local names = { "Alice", "Bob", "Jack" }
table.remove(names, 1)  -- 移除索引为 1 的元素（也可以认为是移除开头）
for key, value in pairs(names) do
    print(key, value)
end

-- 1       Bob
-- 2       Jack
```

## 排序 `table.sort`

简单排序。

```lua
local nums = { 5, 3, 2, 4, 1 }
table.sort(nums)
for key, value in pairs(nums) do
    print(value)
end

-- 1 2 3 4 5
```

如果表中嵌套了表，还可以根据表的字段进行排序，需要给 `table.sort` 传入一个 function 作为参数，在函数内判断字段大小。

```lua
local children =
{
    {
        name = "Alice",
        age = 5
    },
    {
        name = "Bob",
        age = 2
    },
    {
        name = "Jack",
        age = 0
    }
}
table.sort(children, function (a, b)
    -- age 从小到大排序
    if a.age < b.age then
        return true
    end
    return false
end)
for key, value in pairs(children) do
    print(value.name, value.age)
end

-- Jack    0
-- Bob     2
-- Alice   5
```

## 拼接 `table.concat`

可以把表中的所有元素拼接成一个字符串，可以指定分隔符，拼接的元素范围。

```lua
local names = { "Alice", "Bob", "Jack" }
local str = table.concat(names)  -- 直接拼接
print(str)

-- AliceBobJack

local names = { "Alice", "Bob", "Jack" }
local str = table.concat(names, ",")  -- 用逗号隔开
print(str)

-- Alice,Bob,Jack

local names = { "Alice", "Bob", "Jack" }
local str = table.concat(names, ",", 2, 3)  -- 只拼接索引为 2 和 3 的元素
print(str)

-- Bob,Jack
```
