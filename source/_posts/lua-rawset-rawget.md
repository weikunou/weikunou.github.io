---
title: Lua 绕过元表
date: 2024-06-23 11:26:30
tags: Lua
---

Lua 绕过元表，直接访问 table 的字段。

<!--more-->

# 绕过元表

`rawset(table, index, value)`，在不触发元方法的情况下，设置 `table[index]` 的值为 value。

`rawget(table, index)`，在不触发元方法的情况下，获取 `table[index]` 的值。

现有一个 hero 表，设置了元表和元方法。

```lua
local hero = {
    level = 1,
    exp = 0
}

local mt = {
    __index = function (table, key)
        print("trigger __index")
    end,
    __newindex = function (table, key, value)
        print("trigger __newindex")
    end
}
setmetatable(hero, mt)
```

如果去访问一个不存在的 key，则会进入元表的元方法。

因为 `__newindex` 被赋值为一个函数，只有打印，没有把值存到 table 里，所以 hero 里还是没有 star 字段。

```lua
hero.star = 5
print(hero.star)

-- trigger __newindex
-- trigger __index
-- nil
```

现在使用 rawset 为 hero 添加一个新字段 star，再使用 rawget 从 hero 中获取 star 字段。

可以发现，没有触发元方法的打印，hero 表中也添加了 star 字段。

```lua
rawset(hero, "star", 5)
local star = rawget(hero, "star")
print(star)

-- 5
```

# 防止死循环

修改一下元方法，在 `__index` 中返回 `table[key]` 的值，在 `__newindex` 中设置 `table[key]` 为 value。

```lua
local hero = {
    level = 1,
    exp = 0
}

local mt = {
    __index = function (table, key)
        print("trigger __index")
        return table[key]
    end,
    __newindex = function (table, key, value)
        print("trigger __newindex")
        table[key] = value
    end
}
setmetatable(hero, mt)
```

看上去似乎没有什么问题，但是如果触发了这两个元方法，它们内部的逻辑（也就是 `table[key]`）又会触发元方法，进入死循环，最后栈溢出。

```lua
hero.star = 5

-- trigger __newindex
-- trigger __newindex
-- trigger __newindex
-- ...
-- C stack overflow

print(hero.star)

-- trigger __index
-- trigger __index
-- trigger __index
-- ...
-- C stack overflow
```

要防止这种死循环，可以利用 rawset 和 rawget，避免再次触发元方法。

```lua
local hero = {
    level = 1,
    exp = 0
}

local mt = {
    __index = function (table, key)
        print("trigger __index")
        return rawget(table, key)
    end,
    __newindex = function (table, key, value)
        print("trigger __newindex")
        rawset(table, key, value)
    end
}
setmetatable(hero, mt)
```

再次访问 star 字段，就不会栈溢出了。

```lua
hero.star = 5
-- trigger __newindex

print(hero.star)
-- 5
```
