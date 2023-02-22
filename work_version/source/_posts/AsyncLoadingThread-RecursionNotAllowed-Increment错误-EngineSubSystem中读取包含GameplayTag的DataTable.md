---
title: >-
  [Crash]AsyncLoadingThread.RecursionNotAllowed.Increment错误
date: 2023-02-22 10:08:29
categories:
- Client领域
- 引擎
- Unreal
- Crash
tags:
- UE
- Crash
---
### 1. 结论

- 问题: 读取DataTable时报AsyncLoadingThread.RecursionNotAllowed.Increment错误
- 原因: DataTable中包含GameplayTag, 且使用EngineSubSystem::Initialize来进行初始化, 会导致该错误
- 解决方案:  使用字符串来初始化加载相关GameplayTag配置
- 待后续调研: 
  - 异步加载为何会导致recursion
  - 排除package id重复, module加载顺序错误



### 2. 错误与原因

在运行Development版本是(不论是windows还是mobile)运行时都会报AsyncLoadingThread.RecursionNotAllowed.Increment()错误. 但是运行Shipping与Testing版本就没有该错误, 且一切正常.

<img src="https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/ue4_async_recursion.png" tyle="zoom:50%;" />

代码crash处如下, 也是为啥只有Development版本有问题. 

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/async_loading_recursion_code.png)

在AsyncPackageLoader中使用了全局的GPackageLoader来调用所有的FlushLoading, 因此上面图中的This指针是一样的. 导致内部assert.

根据callstack上移到FlushLoading中, 已经有代码排除了相同PackageID, 可以排除相同PackageID导致 

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230222144953.png)

对UE的异步加载系统了解较少,  这里留有对异步加载的整体疑问, 已经知道是一部加载package的问题, 通过Callstack可以看到是DataTable的读取出现问题. 排查后发现EngineSubSystem::Initialize中读取包含GameplayTag的DataTable, 结构如下, 这么做的目的时, 全局使用该TableRow中的成员变量来比对tag而不是字符, 这样对gameplaytag的修改, 不会造成任何蓝图或者代码的错误, 导致需要全局修改.

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230222150138.png)

怀疑读取Table时, GameplayTag的Module还未初始化. 调研整体UE中Module的初始化流程, GameplayTag模块属于引擎相关, 在GEngineLoop.PreInit中已经读取, 而项目工程在根目录下的.uproject配置default读取阶段, 会在GEngineLoop.Init()中读取, 因此可以保障此时GameplayTag模块已经加载完毕. 模块详细加载的文档可以查看这份: https://ue5wiki.com/wiki/24007/

暂时只能猜测是Module加载时异步加载还没有完全启动, 导致递归, 因此整个表格的读取我们暂时回归硬代码如下

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230222155158.png)
