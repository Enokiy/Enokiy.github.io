---
title: 南京大学《软件分析》学习笔记
authors: Enokiy
date: 2024-03-29
categories: [安全]
tags: [静态分析]
---

## 概述
本文主要内容是南京大学《软件分析》网课的学习笔记，视频来自https://space.bilibili.com/2919428/channel/series 。

## 简介

![](/assets/images/static-program-analysis/2024-04-02-09-12-05.png)

* Programming Languages: 
    * 理论: 语言的语法、语义、类型系统等形式化,然后在其形式化基础上用理论方法证明该语言的诸多性质
    * 环境: 主要包括编译系统和运行时系统两个部分
    * 应用: 程序分析、程序验证、程序合成等
* 静态分析: 在不运行一个程序的情况下，通过某种方法分析该程序就知道其是否满足某些性质。（Static analysis analyzes a program P to reason about its behaviors and determines whether it satisfies some properties before running P.）
* Soundness 、Completeness、误报、漏报：
* 静态分析，实际上就是在确保（或尽可能接近）soundness的前提下，在分析精度与分析速度之间取得一个有效的平衡。

![](/assets/images/static-program-analysis/2024-04-02-15-03-31.png)

* Abstraction + overapproximation：
> 抽象是指对数据的抽象;值域、抽象值域
> overapproximation：包括转换函数（Transfer function）和控制流
* 转换函数：在程序分析中，给定一个程序语句，转换函数定义如何根据该“语句的语义”和“分析任务”来过近似该语句的运行时行为。
    * 转换函数决定应如何评价不同程序在抽象值域上的状态。（In static analysis, transfer functions define how to evaluate different program statements on abstract values.）
* 控制流：程序控制流汇聚处的抽象数据合并（merging）
* 程序控制流汇聚处的抽象数据合并
* 从程序控制流图的视角总结一下过近似：如果将一个程序表达成控制流图，那么它无非是由节点（语句）和边（控制流）构成，如果我们将节点和边都过近似了（前者通过转换函数，后者通过控制流汇聚处的抽象数据合并处理），我们就将整个程序过近似了。


## IR

###  术语
* AST：抽象语法树
* IR：Intermediate Representation
* IR: Three-Address Code (3AC)
* Static Single Assignment (SSA)
* Basic Blocks (BB)
* Control Flow Graphs (CFG)

### 编译器与静态分析

#### 编译器

* 词法分析、语法分析、语义分析

![](/assets/images/static-program-analysis/20240410090041.png)

* AST vs IR：

|AST|IR|
|--|--|
|更类似语法结构|更类似机器码|
|语言强相关|语言无关|
|适用于类型检查|简洁、通用|
|缺少控制流信息|包含控制流信息|
||静态分析的基础|

![](/assets/images/static-program-analysis/20240410090703.png)

#### IR(中间表示)
* 三地址码: name、constant、Compiler-generated temporary
* 常见的三地址码形式：

![](/assets/images/static-program-analysis/20240410092034.png)


* java虚拟机 jvm指令：

  | 指令            | 说明                                                         |
  | --------------- | ------------------------------------------------------------ |
  | invokeinterface | 用以调用接口方法，在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用。（Invoke interface method） |
  | invokevirtual   | 指令用于调用对象的实例方法，根据对象的实际类型进行分派（Invoke instance method; dispatch based on class） |
  | invokestatic    | 用以调用类方法（Invoke a class (static) method ）            |
  | invokespecial   | 指令用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法。（Invoke instance method; special handling for superclass, private, and instance initialization method invocations ） |
  | invokedynamic   | JDK1.7新加入的一个虚拟机指令，相比于之前的四条指令，他们的分派逻辑都是固化在JVM内部，而invokedynamic则用于处理新的方法分派：它允许应用级别的代码来确定执行哪一个方法调用，只有在调用要执行的时候，才会进行这种判断,从而达到动态语言的支持。(Invoke dynamic method) |

#### Static Single Assignment (SSA)

* 每个变量都有一个新的名字
* 将新名称传播到后续使用
* 每个变量只有一个定义

#### Control Flow Analysis
* 控制流分析用于构建控制流图
* CFG是静态分析的基本结构
* CFG中的节点可以是三地址码，也可以是basic block。
* basic block：
* BB
>* 入口唯一,出口唯一
>* 构造BB的方法：

![build BB](/assets/images/static-program-analysis/20240516085855.png)

* CFG
>* 控制流图中的节点是BB
>* block A到block B之间存在一条边当且仅当:
>>* block A的结尾有一个条件/非条件跳转到B;
>>* 原始指令顺序中B紧跟着A并且A不是以非条件跳转结束的?


## DFA-AP


## DFA-FD


## inter

## Inter

## PTA

## PTA-FD

## PTA-CS

## Security

## IFDS

## Soundiness

## Datalog

## introduction













 



































































