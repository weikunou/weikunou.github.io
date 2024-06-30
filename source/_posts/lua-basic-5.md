---
title: Lua 基础 05 时间
date: 2024-06-09 10:30:51
tags: Lua
---

Lua 基础相关知识 第五期（完结）

<!--more-->

# 时间戳

通过 `os.time` 函数，可以获取当前的时间戳。

```lua
local time = os.time()
print(time)

-- 1717895996 -> 2024-06-09 09:19:56
```

如果要指定日期，获取对应的时间戳，可以往 `os.time` 函数传入一个表，设定日期。

```lua
local date = {
    year = 2024,  -- 年份
    month = 6,    -- 月份
    day = 9,      -- 天数
    hour = 9,     -- 小时
    min = 30,     -- 分钟
    sec = 30      -- 秒数
}
local time = os.time(date)
print(time)

-- 1717896630 -> 2024-06-09 09:30:30
```

注意到，如果只写了年份、月份、天数，没有小时、分钟、秒数，默认是当天中午 12 点。

```lua
local date = {
    year = 2024,  -- 年份
    month = 6,    -- 月份
    day = 9,      -- 天数
}
local time = os.time(date)
print(time)

-- 1717905600 -> 2024-06-09 12:00:00
```

# 日期

通过 `os.date` 函数，可以把时间戳转换成有格式的日期。

```lua
local time = os.time()
print(time)

-- 1717896843


local date = os.date("%Y/%m/%d %H:%M:%S", time)
print(date)

-- 2024/06/09 09:34:03
```

如果不带任何参数调用 `os.date`，则获取当前时间戳对应的默认日期格式。

```lua
local date = os.date()
print(date)

-- Sun Jun  9 09:35:58 2024
```

有时候，我们需要获取日期当中的某个参数，比如天数，可以把 os.date 的第一个参数改成 `"*t"`，此时它会返回一个表。

```lua
local time = os.time()
print(time)

-- 1717897482


local dateTable = os.date("*t", time)
print(dateTable)

-- table: 000001736C002770


for key, value in pairs(dateTable) do
    print(key, value)
end

-- year    2024
-- wday    1
-- day     9
-- min     44
-- month   6
-- sec     42
-- isdst   false
-- yday    161
-- hour    9
```

如果要计算多少天后的日期，可以给这个日期表中的 day 字段加上一定的天数，转成时间戳后，再格式化日期。

```lua
-- 获取当前时间戳
local time = os.time()
-- 转换成日期格式
local currentDate = os.date("%Y/%m/%d %H:%M:%S", time)
print(currentDate)

-- 2024/06/09 09:51:20


-- 把当前时间戳转成一个表，天数加上 100 天
local dateTable = os.date("*t", time)
dateTable.day = dateTable.day + 100


-- 获取 100 天后的时间戳
local future = os.time(dateTable)
-- 转换成日期格式
local futureDate = os.date("%Y/%m/%d %H:%M:%S", future)

print(futureDate)

-- 2024/09/17 09:51:20
```

# 时间差

通过 `os.difftime` 函数，计算两个时间戳之间的差值。

例如，计算一下当前时间到次日零点的秒数。

```lua
-- 获取当前时间戳
local now = os.time()
print(os.date("%Y/%m/%d %H:%M:%S", now))

-- 2024/06/09 10:12:39


-- 把当前时间戳转成一个表，天数加 1，小时、分钟、秒数清零
local nowDate = os.date("*t", now)
nowDate.day = nowDate.day + 1
nowDate.hour = 0
nowDate.min = 0
nowDate.sec = 0


-- 计算次日零点时间戳
local future = os.time(nowDate)
print(os.date("%Y/%m/%d %H:%M:%S", future))

-- 2024/06/10 00:00:00


-- 计算两个时间戳的差值
local diff = os.difftime(now, future)
print(diff)

-- -49641.0
```

# CPU 时间

通过 `os.clock` 函数，可以获取消耗的 CPU 时间，可以用在性能测试。

例如，计算一下 for 循环累加 1 亿次消耗的时间。

```lua
-- 开始前
local before = os.clock()
print(before)  -- 0.081

-- 累加 1 亿次
local sum = 0
for i = 1, 100000000 do
    sum = sum + i
end

-- 结束后
local after = os.clock()
print(after)   -- 0.445

-- 经过的时间
local diff = after - before
print(diff)    -- 0.364
```
