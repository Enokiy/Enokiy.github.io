---
title: lua安全-源码静态分析
authors: Enokiy
date: 2024-02-02
categories: [lua]
tags: [源码静态分析]
---

## lua基础语法

lua基础知识参考菜鸟教程等，这里只记录一些lua中特殊的点：
1. 可以用 2 个方括号 "[[]]" 来表示"一块"字符串：

```lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```

2. #string #table 返回字符串或表的长度

3. 字符串长度计算： 在 Lua 中，要计算字符串的长度（即字符串中字符的个数），你可以使用 string.len函数或 utf8.len 函数，包含中文的一般用 utf8.len，string.len 函数用于计算只包含 ASCII 字符串的长度。

```lua
local myString = "Hello,世界!"

-- 计算字符串的长度（字符个数）
local length1 = utf8.len(myString)
print(length1) -- 输出 9

-- string.len 函数会导致结果不准确
local length2 = string.len(myString)
print(length2) -- 输出 13
```

4. string.char(arg) 和 string.byte(arg[,int])  char 将整型数字转成字符并连接， byte 转换字符为整数值(可以指定某个字符，默认第一个字符)。

```lua
> string.char(97,98,99,100)
abcd
> string.byte("ABCDE",1,4)  
65      66      67      68
```

5. lua 索引默认从 1 开始

6. Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

```lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```

7. lua module加载机制： require用于搜索Lua文件的路径是存放在全局变量package.path中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始化这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。如果找到目标文件，则会调用 package.loadfile 来加载模块。否则，就会去找 C库。C库搜索的文件路径是从全局变量 package.cpath获取，而这个变量则是通过环境变量 LUA_CPATH 来初始化。

8. 

## lua 源码审计