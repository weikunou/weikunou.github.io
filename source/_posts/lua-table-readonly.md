---
title: Lua 只读表
date: 2024-06-16 15:26:28
tags: Lua
---

Lua 的 table 建立只读机制，保护 table 不能被随意修改。

<!--more-->

# 建立只读机制

Lua 的 table 通常情况下是可以随意修改字段的值，或者新增字段。

如果想要建立只读机制，保护表只能读取，而不能被随意修改，可以利用元表。

# 禁止新增字段

首先，利用 `__newindex`，可以禁止新增字段。

例如，现在有一个 student 表，包含 name 和 age。

定义一个 mt 表，`__newindex` 打印只读提示。

给 student 表设置元表 mt。

```lua
local student = {
    name = "Alice",
    age = 18
}

local mt = {
    __newindex = function ()
        print("Can not modify a readonly table")
    end
}

setmetatable(student, mt)
```

现在可以读取字段，但是不能新增字段了。

例如给 student 增加一个 score 字段，但是会打印只读提示，没有赋值，尝试打印 score 字段会输出 nil。

```lua
local name = student.name
print(name)  -- Alice

student.score = 0     -- Can not modify a readonly table
print(student.score)  -- nil
```

# 禁止修改字段

经过上述步骤后，虽然不能新增字段，但还是可以对已有的字段进行修改。

```lua
student.age = 20
print(student.age)  -- 20
```

Lua 没有提供修改字段的元方法，不过我们可以利用空表，把修改字段的行为，转化成新增字段的行为。

因为对于空表来说，访问任何字段都是不存在的，都会是新增字段的行为。

修改一下之前的代码，定义一个 empty 表，给 empty 设置元表 mt，元表里定义 `__index`，赋值为 student。

```lua
local student = {
    name = "Alice",
    age = 18
}

local empty = {}
local mt = {
    __index = student,
    __newindex = function ()
        print("Can not modify a readonly table")
    end
}
setmetatable(empty, mt)
```

此时，访问 empty 表，是不能修改字段值的。

```lua
empty.age = 20    -- Can not modify a readonly table
print(empty.age)  -- 18
```

不过，此处仍然可以直接去修改 student 表，所以需要对这套方法做一个封装。

# 封装只读函数

定义一个全局函数 Readonly，把上述的步骤放到函数中，接收一个 table，返回经过处理的空表。

```lua
Readonly = function (table)
    local empty = {}
    local mt = {
        __index = table,
        __newindex = function ()
            print("Can not modify a readonly table")
        end
    }
    setmetatable(empty, mt)
    return empty
end

local student = {
    name = "Alice",
    age = 18
}
student = Readonly(student)
```

再尝试一下修改 student，已经不能修改字段的值了。

```lua
student.age = 20    -- Can not modify a readonly table
print(student.age)  -- 18
```

# 嵌套表

上面的只读函数只能处理一层表，如果表里面还嵌套了表，那么嵌套的表还是非只读的。

此时可以递归检查表里是否有嵌套表。

```lua
Readonly = function (table)
    -- 遍历每个字段
    for key, value in pairs(table) do
        -- 有嵌套表
        if type(value) == "table" then
            -- 递归设置只读
            table[key] = Readonly(value)
        end
    end
    -- 设置只读的步骤
    local empty = {}
    local mt = {
        __index = table,
        __newindex = function ()
            print("Can not modify a readonly table")
        end
    }
    setmetatable(empty, mt)
    return empty
end
```

在 student 表里新增一个 score 表，包含 math 和 english 字段。

经过只读函数递归处理之后，score 表也是只读的。

```lua
local student = {
    name = "Alice",
    age = 18,
    score = {
        math = 100,
        english = 100
    }
}
student = Readonly(student)

student.score.math = 20    -- Can not modify a readonly table
print(student.score.math)  -- 100
```
