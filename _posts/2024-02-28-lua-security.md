---
title: lua安全-源码审计
authors: Enokiy
date: 2024-02-02
categories: [lua]
tags: [源码审计]
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

8. Lua 协同程序(coroutine)

| 方法 |	描述 |
|--|--|
| coroutine.create()|	创建 coroutine，返回 coroutine， 参数是一个函数，当和 resume 配合使用的时候就唤醒函数调用 |
| coroutine.resume()|	重启 coroutine，和 create 配合使用
| coroutine.yield()	|   挂起 coroutine，将 coroutine 设置为挂起状态，这个和 resume 配合使用能有很多有用的效果 |
| coroutine.status()|	查看 coroutine 的状态 *注：coroutine 的状态有三种：dead，suspended，running，具体什么时候有这样的状态请参考下面的程序* |
| coroutine.wrap（）|	创建 coroutine，返回一个函数，一旦你调用这个函数，就进入 coroutine，和 create 功能重复 |
| coroutine.running()|	返回正在跑的 coroutine，一个 coroutine 就是一个线程，当使用running的时候，就是返回一个 coroutine 的线程号 |

9. LUA中面向对象的开发可以通过 table + function 模拟类实现
10. 数组索引从1开始
11. 逻辑假False: lua中只有nil和false表示逻辑假，0也表示逻辑真。

## lua 源码审计
### 命令注入
用户可控的输入未经校验或者未经正确校验就拼接到以下API的参数中执行，可能导致命令注入漏洞：

|API |说明 |
|--|--|
|os.execute ([command])| 等价于C函数system。 它调用系统解释器执行 command。 如果命令成功运行完毕，第一个返回值就是 true， 否则是 nil。|
|io.popen (prog [, mode])|用一个独立进程开启程序 prog， 返回的文件句柄可用于从这个程序中读取数据 （mode为 "r"时，这是默认值） 或是向这个程序写入（当 mode 为 "w" 时）|
|||

### 代码注入

|API |说明 |
|--|--|
|require (modname)|加载一个模块。 这个函数首先查找 package.loaded 表， 检测 modname 是否被加载过。 如果被加载过，require 返回 package.loaded[modname] 中保存的值。 否则，它试着为模块寻找 加载器 。不过一般情况下应该不会有人写代码允许require的参数是用户可控的吧？|
|load (chunk [, chunkname [, mode [, env]]])|加载一个代码块。如果 chunk 是一个字符串，代码块指这个字符串。 如果 chunk 是一个函数， load 不断地调用它获取代码块的片断。 如果chunk是一个字符串,并且输入用户可控的话，可以插入恶意代码导致安全风险。|
|loadfile ([filename [, mode [, env]]])|和load 类似， 不过是从文件 filename 或标准输入（如果文件名未提供）中获取代码块。只加载，不执行lua代码|
|dofile ([filename])|打开该名字的文件，并执行文件中的 Lua 代码块。 不带参数调用时， dofile 执行标准输入的内容（stdin）。 返回该代码块的所有返回值。 |
|loadstring (string [, chunkname]) |和load类似，lua5.1之后废弃了该API。|
|package.cpath|用于初始化lua加载模块时查找的C库的路径|
|package.path|用于初始化lua加载模块时查找的lua文件的路径|
|package.loadlib (libname, funcname)|宿主程序动态链接 C 库 libname 。和 require 不同， 它不会做任何路径查询，也不会自动加扩展名。 libname 必须是一个 C 库需要的完整的文件名，如果有必要，需要提供路径和扩展名。|
|package.searchpath (name, path [, sep [, rep]])|在指定 path 中搜索指定的 name（function name ？还是module name？ ）|
|||

```lua
---load 
param = "aaa"
f = load("print('"..param.."')") --- 本意是print(param)
f()  

param1 = "bbb')\nos.execute('dir" --- 通过字符串拼接实现恶意代码执行
load("print('"..param1.."')")()

```

### 文件操作
不管什么语言都有的安全风险，在文件操作过程中如果对文件路径处理不当可能导致任意文件读/写，相关的API：

|API |说明 |
|--|--|
|io.open (filename [, mode])|以mode方式打开文件，mode可以为"："r"读、"w"写、"a"追加、"r+":read update mode、 "w+": write update mode 、"a+": append update mode,|
|os.remove (filename)|删除文件|
|os.rename (oldname, newname)|文件重命名，可能的安全风险：跨目录文件移动？|
|io.output ([file])||
|io.input ([file])||
|io.read (···)|等价于 io.input():read(···)|
|io.write (···)|等价于 io.output():write(···)|
|file:read (···)|文件读,file是使用io.open打开的文件句柄|
|file:write (···)|文件写,file是使用io.open打开的文件句柄|
|||

### 其他
其他类型的安全问题需要结合业务具体分析，比如如果提供了http请求类业务，那需要考虑SSRF、SQL注入等问题；如果是与C语言结合使用，需要考虑溢出等安全风险。
后续再更深入的分析lua虚拟机等知识。

再记录下lua反编译:
使用[unluac](https://github.com/HansWessels/unluac)可以很方便的把编译后的luac文件反编译回来，如下图是个编译后的lua二进制文件:

![](/assets/images/lua/2024-04-03-18-08-00.png)

使用unluac进行反编译：

> java -jar unluac.jar demo > decompiled.lua

![](/assets/images/lua/2024-04-03-18-16-07.png)
## 参考
* https://www.runoob.com/lua/lua-tutorial.html
* https://www.lua.org/manual/5.4/
* https://www.runoob.com/manual/lua53doc/