---
title: Lua 基础 01 入门
date: 2024-05-12 14:41:45
tags: Lua
---

Lua 基础相关知识 第一期

<!--more-->

# 注释

```lua
--  单行注释

--[[
    多行注释
--]]

-- 多加一个横杠符号就能重新启用注释内的代码
---[[
    print("Lua")
--]]
```

# 数据类型

Lua 是动态类型语言，变量不需要类型定义，只需要为变量赋值。

Lua 有 8 种基本类型：

1. nil（表示一个无效值，条件表达式中表示 false）
2. boolean（false 或 true，0 也是 true）
3. number（表示双精度类型的实浮点数）
4. string（字符串，由一对单引号或双引号包括，也可以用两个方括号）
5. table（表，是一个关联数组，数组的索引可以是数字、字符串或表类型）
6. function（函数）
7. userdata（表示任意存储在变量中的 C 语言数据结构）
8. thread（表示执行的独立线程，用于执行协同程序）

可以使用 type() 函数查看数据类型。

```lua
print(type(nil))     -- nil
print(type(true))    -- boolean
print(type(1))       -- number
print(type("Lua"))   -- string
print(type({}))      -- table
print(type(print))   -- function
```

## 布尔

false 或 true，但需要注意，Lua 把除了 false 和 nil 以外的值都视为 true，例如 0 和 `""` 都是 true。

逻辑运算符 and，or，not 的使用：

1. and，如果第一个操作数为 false，就返回第一个操作数，否则返回第二个操作数。
2. or，如果第一个操作数为 true，就返回第一个操作数，否则返回第二个操作数。
3. not，取反。

and 和 or 都是短路求值，只有在必要时才会计算第二个操作数。

常见的写法：

```lua
a = a or 0  -- 如果 a 不存在，则使用默认值 0

a = x < y and x or y  -- 类似三目运算符，但不完全是
-- 操作过程 --

-- 若 x < y 为 true，则 ( x < y and x ) 返回 x
-- 接着 x or y，若 x 存在则为 true，返回 x ( 如果此时 x 不存在或 false，则返回 y，这里是跟三目运算符的区别)

-- 若 x < y 为 false，则 ( x < y and x ) 返回 x < y 的结果 false
-- 接着 false or y，返回 y

not nil    -- true
not false  -- true
not true   -- false
not 0      -- false
```

## 字符串

在对一个数字字符串进行算术运算时，Lua 会尝试将字符串转成数字。

其他语言，例如 C#，是把数字转成字符串进行连接，这里要注意 Lua 的处理方式。

```lua
print("1" + 2)  -- 3
```

而连接字符串则使用 .. 符号。

```lua
print("How " .. "are " .. "you")  -- How are you
```

计算字符串长度用 # 符号。

```lua
print(#"hello")  -- 5
```

字符串可以使用两个方括号表示。

```lua
html = [[
<html>
<head></head>
<body>
    <p>Lua</p>
</body>
</html>
]]

print(html)

--[[ 打印结果也有换行

<html>
<head></head>
<body>
    <p>Lua</p>
</body>
</html>

--]]
```

如果字符串的内容也有两个方括号，就会有语法错误，此时可以在最外围的两对方括号中间添加 = 号。

```lua
html = [=[
<html>
<head></head>
<body>
    <p>Lua 这里多出的 ]] 会和 [[ 匹配，后面的内容就不被包括在字符串内了</p>
</body>
</html>
]=]
```

## 表

表可以看成是数组或哈希表，取决于 key 值是从 1 开始的连续正整数还是其他类型。

如果没有指定 key 值，默认就是数组。

不同于其他语言，Lua 的数组索引是从 1 开始的。

```lua
table = { "a", "b", "c" }
print(table[1])  -- a
print(table[2])  -- b
print(table[3])  -- c
```

指定 key 值，就变成了哈希表。可以使用 [] 或 . 的方式获取 value 值。

```lua
table = {
    ["name"] = "Alice",
    ["age"] = 18,
}
print(table["name"])  -- Alice
print(table.age)      -- 18
```

需要注意的是，如果直接给表赋值第三个位置的值，那么前两个位置的值会自动填充为 nil。

```lua
table = {
    [3] = "Alice",
}
print(table[1])  -- nil
print(table[2])  -- nil
print(table[3])  -- Alice
```

计算表的长度也是用 # 符号，前提是数组的形式，哈希表不适用（只能计算 key 值为连续正整数的部分）。

```lua
table = { "a", "b", "c" }
print(#table)  -- 3

table = {
    [1] = "a",
    [3] = "c"
}
print(#table)  -- 1

table = {
    [1] = "a",
    ["2"] = "b",  -- key 值为字符串
    [3] = "c"
}

print(#table)  -- 1
```

如果要判断表是否为空，可以使用 next() 函数，获取下一个键值对。

```lua
table = {}
print(next(table) == nil)  -- true

table = { "a" }
print(next(table) == nil)  -- false
```

## 函数

函数使用 function 定义，指定函数名和参数，end 结尾。

```lua
function Add(a, b)
    return a + b
end

print(Add(1, 2))  -- 3
```

函数可以保存到变量中，也可以保存到表中。

```lua
add = function (a, b)
    return a + b
end
print(add(1, 2))  -- 3

table = {
    ["add"] = add
}
print(table["add"](1, 2))   -- 3
print(table.add(1, 2))      -- 3
```

# 变量

Lua 中声明变量默认是全局变量，只有在变量前添加 local 关键字才是局部变量。

```lua
a = 1         -- 全局变量
local b = 2   -- 局部变量
```

可以同时对多个变量进行赋值。

```lua
a, b = 5, 6
print(a, b)  -- 5 6
```

赋值前，Lua 会先计算右边的值再赋值，所以可以简便地进行变量的交换。

```lua
a, b = 5, 6
a, b = b, a
print(a, b)  -- 6 5
```

当变量的个数和值的个数不同时，Lua 会采取以下策略：

- 变量的个数 > 值的个数：补足 nil
- 变量的个数 < 值的个数：多余的值被忽略

```lua
a, b, c = 0
print(a, b, c)  -- 0 nil nil

a = 0, 1, 2
print(a)  -- 0
```

# 分支循环

## 分支

分支结构如下，每个 if 后面要接 then，最后以 end 结尾。

```lua
local score = 60
if score < 60 then
    print("不及格")
elseif score < 80 then
    print("及格")
elseif score < 100 then
    print("优秀")
else
    print("超标")
end

-- 及格
```

## 循环

循环结构如下，指定 i 的初始值、终止值、步长。

```lua
for i = 1, 5, 1 do
    print("i = " .. i)
end

-- i = 1
-- i = 2
-- i = 3
-- i = 4
-- i = 5
```

## 遍历表

有三种方法遍历表，常规的第一种如下：

```lua
table = { "a", "b", "c" }

-- #table 获取 table 的长度
for i = 1, #table, 1 do
    print(table[i])
end

-- a
-- b
-- c
```

第二种是使用 pairs 方法：

```lua
table = { "a", "b", "c" }

for key, value in pairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 1 value = a
-- key = 2 value = b
-- key = 3 value = c
```

第三种是使用 ipairs 方法：

```lua
table = { "a", "b", "c" }

for key, value in ipairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 1 value = a
-- key = 2 value = b
-- key = 3 value = c
```

## pairs 和 ipairs 的区别

pairs，遍历所有键值对，顺序随机，可以返回 nil。

ipairs，从 key 值为 1 开始顺序遍历，key 值不连续则会停止，不能返回 nil。

例 1，表中有两个键值对，缺失了 key 为 2 的值。

pairs 会遍历所有键值对，ipairs 只会遍历第一个键值对，因为找不到 key 为 2 的值，就停止了。

```lua
table = {
    [1] = "a",
    [3] = "c"
}

for key, value in pairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 1 value = a
-- key = 3 value = c

for key, value in ipairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 1 value = a
```

例 2，补上了一个键值对，但是 key 值是字符串 "2"。

pairs 依然会遍历所有键值对，但是顺序乱了。ipairs 依然只能遍历第一个键值对。

```lua
table = {
    [1] = "a",
    ["2"] = "b",  -- key 值为字符串
    [3] = "c"
}

for key, value in pairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 3 value = c
-- key = 1 value = a
-- key = 2 value = b

for key, value in ipairs(table) do
    print("key = " .. key .. " value = " .. value)
end

-- key = 1 value = a
```
