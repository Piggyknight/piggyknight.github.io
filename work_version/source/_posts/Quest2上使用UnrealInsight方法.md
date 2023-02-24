---
title: >-
  [工具-性能]Quest2上使用UnrealInsight的方法
date: 2023-02-23 20:42:13
categories:
- Client领域
- 引擎
- Unreal
- 性能优化
- VR
tags:
- UE
- 性能优化
- 工具
---

### 详细步骤

- 在源代码引擎目录Engine/Build/Android/UE4Game/下获取ue4commandline.txt文件, 文件名大小写无关, 使用时去掉后缀template

  ![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223214440.png)

  - 文件内部格式如下

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223214330.png)

具体参数参考官方文档,剪切如下

> 1. Notes on each dir/flag shown above:
>
> 2. - **<ProjectName>**
>
>    - - Just the name of the Unreal project - this will match the entry in /sdcard/UE4Game/<ProjectName>
>
>    - **<UProjectName>**
>
>    - - The UProjectName will match the output binary (dictated by the engine) so you can just use the APK name or find the .uproject file in your game’s directory.
>
>    - **<MapName>**
>
>    - - This one is optional, but it allows you to customize launch arguments on a per-map basis. If you know the map name, you can just put it here.
>      - Pro Tip: if you have a benchmarking scene that you are using, you can set up tracing for only that map so that the tracing overhead doesn’t affect other profiling on regular maps. This is nice because you can keep that launch arg file in the same location without having to remove or rename it between play types.
>
>    - **-trace**
>
>    - - Allows you to define the channels to tap into for the trace. Here are the combinations that I use when profiling:
>
>      - - **Most Detail/Most Overhead:** log,counters,cpu,frame,bookmark,file,loadtime, gpu,rhicommands,rendercommands,object
>
>        - **Decent Detail/Minimal Overhead:** counters,cpu,frame,bookmark,gpu
>
>        - Additional:
>
>        - - **Stats** (experimental statstrace in 4.26 (Enable in code with #define EXPERIMENTAL_STATSTRACE_ENABLED 1 in Engine/Source/Runtime/Core/Public/Stats/StatsTrace.h)
>          - **LoadTime** - includes ‘Loading - Main Thread’ and ‘Loading - Async Thread’ tracks
>
>    - **-statnamedevents**
>
>    - - When combined with -trace=cpu option this will activate even more CPU timing events
>
>    - **-tracehost**
>
>    - - IP address of the host machine with the Unreal Insights instance running that you want to connect to (127.0.0.1)
>
>    - **-tracefile**
>
>    - - Can define a local path on the device that you want to dump the .trace file to as an alternative to live analysis. I keep mine in a static location and just overwrite it each time to always keep a most recent trace.
>
>      - - Just ‘adb pull /sdcard/UE4Game/MostRecentTraceCapture.utrace’ to grab it. Can also just put it in project dir so you can keep most recent for each UE title you’re working on

其实tracefile不是必须, 实际举例如下

> ../../../NextVR/NextVR.uproject /Game/Maps/Wharf/Demo_Wharf -trace=log,counters,cpu,frame,bookmark,file,loadtime,gpu,rhicommands,rendercommands,object -statnamedevents -cpuprofilertrace -tracehost=127.0.0.1



- 通过adb命令将文件上传至相应的项目目录, 如下图

> adb push .\ue4commandline.txt /sdcard/UE4Game/**Project Name**

![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223212045.png)



- 运行如下命令, 让adb把相应的数据传到这个UnrealInsight监听的端口, 全小写, 否则会报找不到socket的错误

> adb reverse tcp:1980 tcp:1980

- 运行Unreal Insights, 一般在下载到源码引擎目录Engine/Binaries/Win64/下

  ![](https://cdn.jsdelivr.net/gh/Piggyknight/pic_bed/20230223212135.png)

- 勾选Auto选项, 启动Quest 2上的应用, apk版本编译记得一定要使用development版本, 默认shipping版本不支持.



###  参考

https://developer.oculus.com/blog/get-started-with-unreal-insights-on-the-oculus-quest/

https://inu-games.com/2020/01/06/unreal-insights-on-oculus-quest/

https://communityforums.atmeta.com/t5/Unreal-VR-Development/Cannot-connect-Unreal-Insights-with-App-Lab-game-with-Quest-2/td-p/976968
