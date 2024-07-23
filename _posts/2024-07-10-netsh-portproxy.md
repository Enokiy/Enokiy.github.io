---
title: windows下使用netsh实现端口转发
authors: Enokiy
date: 2024-07-10
categories: [常用技巧]
tags: [端口转发]
---

## 端口转发

在Windows系统中，可以使用netsh命令行工具来设置端口转发。端口转发是一种将来自一个端口的网络流量转发到另一个端口的技术。如我需要从主机A连接到主机C，但是两个网络不通，而主机B是可以通A和C的，此时可以使用端口转发将来自主机A的网络流量转发到主机B，然后由主机B转发到主机C。

以下是如何在主机B上设置端口转发的步骤：

1. 以管理员身份打开命令提示符。

使用以下命令查看当前的端口转发规则：

```
netsh interface portproxy show all
```

2. 使用以下命令添加一个新的端口转发规则：

```
netsh interface portproxy add v4tov4 listenaddress=<本地IP地址> listenport=<本地端口> connectaddress=<远程IP地址> connectport=<远程端口> protocol=tcp
```

其中：

* <本地IP地址>是主机B的本地IP地址。
* <本地端口>是您要转发的本地端口。
* <远程IP地址>是主机C的IP地址。
* <远程端口>是主机C上的端口。

例如，要将来自主机A的22端口流量转发到主机C的22端口，可以在主机B上执行以下命令：

```
netsh interface portproxy add v4tov4 listenaddress=ip_of_B listenport=22 connectaddress=ip_of_C connectport=22 protocol=tcp
```

3. 查看添加的端口转发规则是否生效：
```
netsh interface portproxy show all
netstat -ano|findstr 22
```

4. 如果要删除端口转发规则，可以使用以下命令：

```
netsh interface portproxy delete v4tov4 listenaddress=<本地IP地址> listenport=<本地端口> protocol=tcp
```

## 踩坑

使用上面的命令设置了端口转发之后用`netsh interface portproxy show all`可以看到设置的信息，但是使用`netstat` 查看端口并没有打开，最终发现需要启动服务IP Helper：

```
# 使用管理员打开终端
net start iphlpsvc
```