---
title: >-
	[crash]中断点异常 STATUS_BREAKPOINT(0x80000003)
date: 2023-02-23 09:41:02
categories:
- Client领域
- 引擎
- Unreal
- Crash
tags:
- UE
- Crash
---

### 1. 结论与总结

- 消息系统中经常容易发生的循环中重入问题, 因此一定要加入额外的add与remove队列, 避免迭代器失效

- 当发现一种突发错误在默认中没有任何额外信息时, 可以查看下lldb的debuger

  

### 2. 错误表现与原因

游戏运行中, 会突然在如下代码行进入断点

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223095349.png)

查看log与堆栈都是如下的0x80000003错误

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223101209.png)

这个异常即中断指令异常，表示在系统未附加内核调试器时遇到断点或断言, 一般属于代码中发现错误主动中断

- 硬代码中断请求，如：asm int 3
- System.Diagnostics.Debugger.Break（C 35）
- DebugBreak（）（WinAPI）



花了一点时间排除问题, 偶然中发现在Rider中Debugger Output(LLDB)中有明确的提示如下, 迭代器失效了

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223100411.png)

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/lldb_status_breakpoint.png)

结合代码, 即消息系统中, 一个消息触发, 生成一个Actor(也可以是删除), 这个Actor可能也去注册这个相同消息, 这样Listeners队列的迭代器即失效, 属于重入问题. 

也就两种方案, 加锁与额外内存(增加add与remove队列, 避免直接操作队列)
