layout: title
title: 排查CPU使用率始终100%问题
date: 2017-02-24 18:21:08
tags:
- problems
categories:
- problems
---

排查CPU使用率始终100%的线上问题，现象，排查过程，解决方案。
<!-- more -->

### 现象
  收到线上机器单个CPU平均load过高的报警，登陆到机器，命令行执行top命令显示如下：   
```
    [zhangxia@xxxx ~]$ top

    top - 17:38:39 up 58 days,  6:38,  2 users,  load average: 0.66, 0.20, 0.19
    Tasks: 119 total,   2 running, 117 sleeping,   0 stopped,   0 zombie
    Cpu0  : 38.3%us, 59.0%sy,  0.0%ni,  0.3%id,  0.0%wa,  0.0%hi,  2.3%si,  0.0%st
    Cpu1  : 20.7%us, 47.4%sy,  0.0%ni, 31.6%id,  0.0%wa,  0.0%hi,  0.0%si,  0.3%st
    Mem:   1922348k total,  1703128k used,   219220k free,     4300k buffers
    Swap:  2096440k total,        0k used,  2096440k free,   235564k cached

    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                
    8288 zhangxia  20   0 2628m  29m  11m S 98.1  1.6   0:25.24 java          
```
  可以发现进程号为8288的进程CPU使用率为98%，接近100%。
### 解决过程
  #### top找到cpu占用最高的进程号
  已找到进程号为8288.
  #### 查看已找到的进程中哪个线程CPU使用率最高
  在top执行的视图里执行shift+H快捷键，提示“Show threads On”，此时展示的PID为线程ID，可以观察到线程id为19234的线程占用CPU最高。或者使用”Top -Hp 进程号“来观察。
```
    top - 17:47:40 up 58 days,  6:47,  2 users,  load average: 0.71, 0.23, 0.18
    Tasks: 526 total,   2 running, 524 sleeping,   0 stopped,   0 zombie
    Cpu0  : 25.5%us, 44.4%sy,  0.0%ni,  0.0%id,  0.0%wa,  0.0%hi,  1.9%si, 28.2%st
    Cpu1  : 19.9%us, 41.4%sy,  0.0%ni, 37.5%id,  0.0%wa,  0.0%hi,  0.0%si,  1.3%st
    Mem:   1922348k total,  1707684k used,   214664k free,     5740k buffers
    Swap:  2096440k total,        0k used,  2096440k free,   237624k cached

    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                
    19234 zhangxia  20   0 2628m  29m  11m R 97.3  1.6   0:26.50 java
```
  ##### jstack+进程号导出线程堆栈
  执行导出某个进程内的所有线程堆栈信息到文件：
```
    jstack 8288 > stack.8288
```
  堆栈信息导出到了当前文件夹，为stack.8288。
  内容如下：
```
      2017-02-24 18:12:51
    Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.45-b02 mixed mode):

    "Attach Listener" #9 daemon prio=9 os_prio=0 tid=0x00007f8abc001000 nid=0x4b48 waiting on condition [0x0000000000000000]
       java.lang.Thread.State: RUNNABLE

    "Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f8b0407e800 nid=0x4b2a in Object.wait() [0x00007f8af12f9000]
       java.lang.Thread.State: WAITING (on object monitor)
    	at java.lang.Object.wait(Native Method)
    	- waiting on <0x00000006c6c06e80> (a java.lang.ref.ReferenceQueue$Lock)
    	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
    	- locked <0x00000006c6c06e80> (a java.lang.ref.ReferenceQueue$Lock)
    	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
    	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

    "Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f8b0407c000 nid=0x4b29 in Object.wait() [0x00007f8af13fa000]
       java.lang.Thread.State: WAITING (on object monitor)
    	at java.lang.Object.wait(Native Method)
    	- waiting on <0x00000006c6c08370> (a java.lang.ref.Reference$Lock)
    	at java.lang.Object.wait(Object.java:502)
    	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:157)
    	- locked <0x00000006c6c08370> (a java.lang.ref.Reference$Lock)

    "main" #1 prio=5 os_prio=0 tid=0x00007f8b04009800 nid=0x4b22 runnable [0x00007f8b0a8ae000]
       java.lang.Thread.State: RUNNABLE
    	at java.io.FileOutputStream.writeBytes(Native Method)
    	at java.io.FileOutputStream.write(FileOutputStream.java:326)
    	at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
    	at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
    	- locked <0x00000006c6c17678> (a java.io.BufferedOutputStream)
    	at java.io.PrintStream.write(PrintStream.java:482)
    	- locked <0x00000006c6c06f30> (a java.io.PrintStream)
    	at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
    	at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
    	at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
    	- locked <0x00000006c6c06ee8> (a java.io.OutputStreamWriter)
    	at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
    	at java.io.PrintStream.write(PrintStream.java:527)
    	- eliminated <0x00000006c6c06f30> (a java.io.PrintStream)
    	at java.io.PrintStream.print(PrintStream.java:669)
    	at java.io.PrintStream.println(PrintStream.java:806)
    	- locked <0x00000006c6c06f30> (a java.io.PrintStream)
    	at CpuLoadHighTest.main(CpuLoadHighTest.java:4)

      "VM Thread" os_prio=0 tid=0x00007f8b04077000 nid=0x4b28 runnable

      "GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00007f8b0401e800 nid=0x4b24 runnable

      "GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00007f8b04020000 nid=0x4b25 runnable

      "GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00007f8b04022000 nid=0x4b26 runnable

      "GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00007f8b04023800 nid=0x4b27 runnable

      "VM Periodic Task Thread" os_prio=0 tid=0x00007f8b040cc800 nid=0x4b30 waiting on condition

      JNI global references: 9
```
  #### 得到线程号8566的十六进制表示
  命令行执行如下命令：
```
    printf '%x\n' 19234
```
  得到十六进制表示为4b22

  #### 在堆栈日志中查找线程号十六进制表示为4b22的线程堆栈信息
    在堆栈文件中查找4b22，得到如下线程堆栈信息：
```
      "main" #1 prio=5 os_prio=0 tid=0x00007f8b04009800 nid=0x4b22 runnable [0x00007f8b0a8ae000]
     java.lang.Thread.State: RUNNABLE
  	at java.io.FileOutputStream.writeBytes(Native Method)
  	at java.io.FileOutputStream.write(FileOutputStream.java:326)
  	at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:82)
  	at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:140)
  	- locked <0x00000006c6c17678> (a java.io.BufferedOutputStream)
  	at java.io.PrintStream.write(PrintStream.java:482)
  	- locked <0x00000006c6c06f30> (a java.io.PrintStream)
  	at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:221)
  	at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:291)
  	at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:104)
  	- locked <0x00000006c6c06ee8> (a java.io.OutputStreamWriter)
  	at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:185)
  	at java.io.PrintStream.write(PrintStream.java:527)
  	- eliminated <0x00000006c6c06f30> (a java.io.PrintStream)
  	at java.io.PrintStream.print(PrintStream.java:669)
  	at java.io.PrintStream.println(PrintStream.java:806)
  	- locked <0x00000006c6c06f30> (a java.io.PrintStream)
  	at CpuLoadHighTest.main(CpuLoadHighTest.java:4)
```
根绝CpuLoadHighTest.main(CpuLoadHighTest.java:4)可得到问题出现的具体位置，然后分析是否死循环死锁等问题即可解决。   
