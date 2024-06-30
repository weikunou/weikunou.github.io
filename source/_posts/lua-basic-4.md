---
title: Lua 基础 04 模块
date: 2024-06-02 15:30:24
tags: Lua
---

Lua 基础相关知识 第四期

<!--more-->

# require

模块，通常是一个表，表里存储了一些字段和函数，单独写在一个 lua 文件。

例如，这是一个 `tools.lua` 文件，定义了一个局部 tools 表，包含一个 log 函数，可以传入标题和信息，函数内部格式化字符串输出。

最后要 `return tools`，在 require 加载这个模块时才能拿到返回的 tools 表。

```lua
local tools = {}

tools.log = function (title, message)
    print(string.format("[%s] %s", title, message))
end

return tools
```

在其他 lua 文件中加载这个 tools 模块，进行使用。

```lua
local tools = require "tools"
tools.log("console", "hello lua")

-- [console] hello lua
```

标准库也可以通过 require 加载，换个名称。

```lua
local m = require "math"
local n = m.abs(-1)
print(n)

-- 1
```

require 会做已加载的检查，如果已经加载过一个模块了，就不会再重复加载了。

如果要移除原来加载的模块，可以把模块从 `package.loaded` 中删除。

```lua
package.loaded.tools = nil
```

# loadfile dofile require

这三个函数都是用来加载和执行外部 lua 脚本的，不过有一些区别：

- loadfile 加载编译指定的 lua 文件，但不执行文件中的代码，需要手动调用`（返回值是一个函数）`
- dofile 加载编译并执行指定的 lua 文件`（返回值是文件中最后一个表达式的值）`
- require 先检查 `package.loaded` 是否已加载模块，若已加载，则直接返回，若不存在，则编译执行一次，并记录到 `package.loaded（返回值通常是一个表）`

这里所说的执行，是指被加载的 lua 文件中，写在表外面可以被执行的语句，例如在 tools 中添加一行打印，表示欢迎使用这个模块。

```lua
local tools = {}

print("Welcome to use this tools module!")

tools.log = function (title, message)
    print(string.format("[%s] %s", title, message))
end

return tools
```

先尝试使用 loadfile，注意，第一行代码加载的是 `tools.lua`，需要增加 `.lua` 后缀名，第一行并没有输出。

第二行代码打印了 loadfile 的返回值，输出的是一个 function。

第三行代码手动调用了返回的函数，才执行了 `tools.lua` 里面的一行打印。

```lua
local tools = loadfile("tools.lua")
print(tools)  -- function: 0000025E2F7791D0
tools()       -- Welcome to use this tools module!
```

再尝试一下 dofile，同样的，第一行代码加载的是 `tools.lua`，第一行就输出了。

第二行代码打印的返回值是 table，也就是 `tools.lua` 最后一行的 `return tools`。

第三行代码则不是直接调用 tools 了，因为它并不是一个函数。应该调用 `tools.log`。

```lua
local tools = dofile("tools.lua")  -- Welcome to use this tools module!
print(tools)                       -- table: 00000150C5D61D30
tools.log("console", "hello lua")  -- [console] hello lua
```

最后回到 require，注意，第一行代码没有 `.lua` 后缀名，第一行就输出了。

后面两行代码和 dofile 是一致的。

```lua
local tools = require("tools")     -- Welcome to use this tools module!
print(tools)                       -- table: 0000025D08BF13D0
tools.log("console", "hello lua")  -- [console] hello lua
```

如果再次加载 lua 文件，loadfile 依然是需要手动调用，dofile 会再次输出，require 则只输出一次，除非移除已加载的模块。

```lua
local tools_loadfile = loadfile("tools.lua")
local tools_loadfile = loadfile("tools.lua")

local tools_dofile = dofile("tools.lua")  -- Welcome to use this tools module!
local tools_dofile = dofile("tools.lua")  -- Welcome to use this tools module!

local tools_require = require("tools")    -- Welcome to use this tools module!
local tools_require = require("tools")

package.loaded.tools = nil
local tools_require = require("tools")    -- Welcome to use this tools module!
```
