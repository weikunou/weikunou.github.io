---
title: Lua 基础 02 元表
date: 2024-05-19 12:36:06
tags: Lua
---

Lua 基础相关知识 第二期

<!--more-->

# 元表

元表是一种特殊类型，可以附加到另一个表上，控制该表的操作行为。

元表通常会结合元方法一起使用，最常用的元方法是 `__index` 和 `__newindex`，它们的作用就相当于其他语言中的 get 和 set。

- `__index`：当访问表中不存在的 key 时，会进一步查找元表中的 `__index` 元方法，将表和 key 传递进去，再次尝试访问该 key。
- `__newindex`：当对表进行赋值时，如果不存在该 key，也就是新增 key，会调用元表中的 `__newindex` 元方法，将表、key 和 value 传递进去，可定义一些额外逻辑。

其他的元方法还有：

`__add`、`__sub`、`__mul`、`__div`，重载表的算术运算符，对表进行算术运算时，可以自定义算术逻辑。

`__eq`、`__lt`、`__le`，重载表的比较运算符，对表进行逻辑运算时，可以自定义比较逻辑。

要把元表附加到另一个表上，需要使用 `setmetatable` 函数，第一个参数是表，第二个参数是元表。

```lua
local obj = {}   -- 表
local meta = {}  -- 元表
setmetatable(obj, meta)  -- 附加元表
```

接着，可以在元表中定义 `__index` 和 `__newindex` 元方法。

首先尝试 `__index`，为它赋值一个函数，然后尝试访问 obj 表中不存在的 key，会调用 `__index`。

```lua
local obj = {}
local meta = {
    __index = function (table, key)
        print("call __index")
        print(table)
        print(key)
    end
}
setmetatable(obj, meta)

print(obj.name)  -- 访问 obj 表中不存在的 name 字段

-- call __index
-- table: 00000273AA151980
-- name
-- nil
```

接着尝试 `__newindex`，为它赋值一个函数，然后尝试为 obj 表赋值一个不存在的 key，会调用 `__newindex`。

```lua
local obj = {}
local meta = {
    __newindex = function (table, key, value)
        print("call __newindex")
        print(table)
        print(key)
        print(value)
    end
}
setmetatable(obj, meta)

obj.name = "myObj"  -- 赋值 obj 表中不存在的 name 字段

-- call __newindex
-- table: 0000017CCA2B0320
-- name
-- myObj
```

# 面向对象

利用元表的 `__index` 元方法，为表 A 附加一个元表 B，表 A 就会在元表 B 中查找它所没有的操作。

此时可以把表 A 看作对象，表 B 看作类。因为所谓的类，就是已经为对象定义好一系列的属性和方法，对象是以类为模板进行实例化。

下面定义一个 Person 表，它拥有两个函数 New 和 Say，其中 New 函数是用来实例化对象的，可以传入一个已有的表，也可以不传，默认为空表。

对于 `self.__index = self` 这行代码可能会有疑惑，此时 self 指的是 Person，为什么要把 Person 的 `__index` 指向自己呢？因为元表是需要结合元方法一起使用的，只设置了元表是没有作用的，我们需要从元表 Person 中获取 Person 已经定义好的函数，就需要为 `__index` 赋值，直接赋值为 Person 即可。

最后就是调用 `setmetatable` 附加元表，把对象 o 返回。

```lua
Person = {}

function Person:New(o)
    o = o or {}             -- 一个对象
    self.__index = self     -- Person.__index = Person
    setmetatable(o, self)   -- 对象 o 附加元表 Person
    return o                -- 返回对象 o
end

function Person:Say(content)
    print("Person Say : " .. content)
end

local obj = Person:New()    -- 实例化对象
obj:Say("obj")              -- 调用 Say 函数

-- Person Say : obj
```

通过使用 `Person:New` 实例化一个对象，然后让对象调用 Say 函数，传入参数，打印结果。

注意到，这里的函数定义和调用都是使用 : 冒号，其实是 Lua 的一个语法糖，隐式传入 self 参数，放在第一个参数位置。

```lua
function Person:Say(content)

end
-- 等价于
function Person.Say(self, content)

end
```
