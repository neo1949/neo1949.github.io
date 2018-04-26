---
title: 【译】Deadlocks and ANRs
date: 2018-04-26 13:54:56
tags:
categories:
---

原文：[Deadlocks and ANRs](http://zenandroid.io/deadlocks-and-anrs/#ananalogyfordeadlocks)

<!-- more -->

在本文中，我们将看看如何分析实际的应用程序无响应（ANR）跟踪，确定原因（结果是我们正在使用的某个库中存在死锁）并消除它。我们也将借此机会提供关于理解本文所需的死锁和相关计算机科学主题的简短入门。

## 关于互斥和死锁的入门
我们先看看死锁和互斥是什么。熟悉这些主题的人可以跳到“解释ANR痕迹”部分。

请记住，这并不是对这个主题的详尽处理（这是相当复杂和有趣的），而是针对那些大学以外的人而言的简短入门书。

### 什么是互斥
在计算机科学中，互斥 <code>mutex</code>（**mutual exclustion** 的缩写）是一种对象，一次只允许一个线程在代码的关键部分访问共享资源（如文件）。在某些语言中（最引人注目的是Java），它被称为 <code>Lock</code>，我们将交替使用这些术语。

它通常有两种方法：<code>acquire</code> 和 <code>release</code>（有时称为 <code>lock</code> 和 <code>unlock</code>）。操作系统内核保证一旦一个线程成功获得 <code>lock</code>，在第一个线程释放它之前，其他线程不能获取它。尝试这样做会阻止第二个线程，直到 <code>lock</code> 再次可用。因此，如果所有线程都确保只使用调用之间的共享资源来获取和释放它们，则保证轮流使用该资源。

### 互斥的类比
为了更好地理解互斥体，设想一个相似的物理世界模式很有帮助。例如，在一些敏捷团队中，习惯使用 “speaking token”（例如球或白板标记）来随时跟踪谁应该说话。只有持有令牌的人才可以发言。当这个人完成时，令牌被放在桌子上，其他人（不一定按顺序）将其拿起并发送他或她的报告。

互斥体以类似的方式工作：线程类似于团队成员，共享资源是同事关注的，而互斥体则由 “speaking token” 表示。锁定互斥体与拾取令牌相似，而解锁它与将令牌重新放回桌子中间相似。

### 互斥在行为上的表现
让我们考虑两个线程：<code>ThreadA</code> 和 <code>ThreadB</code> 尝试写入日志文件。假设他们正在尝试写入 “Hello world from ThreadA” and “Hello world from ThreadB”.

```java
try {  
    final FileOutputStream fos = new FileOutputStream(File.createTempFile("log", ".txt"));

    Runnable r = new Runnable() {
        @Override
        public void run() {
            try {
                fos.write(("Hello world from " + Thread.currentThread().getName()).getBytes());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    };

    new Thread(r, "ThreadA").start();
    new Thread(r, "ThreadB").start();
} catch (IOException ex) {
    ex.printStackTrace();
}
```

即使没有同步，大多数情况下这也会产生预期的效果。然而有时候，或许一千次运行才会出现一次，你会看到这样的输出：
```
Hello worHello world from ThreadB  
ld from ThreadA  
```

这里发生的是 ThreadB 生成它的输出，而 ThreadA 正在生成它自己的文件，并且文件最终被损坏。解决这个问题的方法是使用一个互斥体（在 Java 中称为 <code>Lock</code>）：
```java
try {  
    final FileOutputStream fos = new FileOutputStream(File.createTempFile("log", ".txt"));
    final Lock fileLock = new ReentrantLock();

    Runnable r = new Runnable() {
        @Override
        public void run() {
            try {
                fileLock.lock();
                // Thread is now guaranteed exclusive access
                fos.write(("Hello world from " + Thread.currentThread().getName()).getBytes());
                fileLock.unlock();
                // Thread no longer has exclusive access
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    };

    new Thread(r, "ThreadA").start();
    new Thread(r, "ThreadB").start();
} catch (IOException ex) {
    ex.printStackTrace();
}
```

有了这个补充，就避免了前面描述的情况。在它可以写入文件之前，线程必须获取互斥锁（锁），如果两者同时尝试这样做，Android Linux 内核保证只有一个从方法调用返回而另一个被阻塞直到第一个释放互斥锁。

### 死锁
虽然互斥体在正确使用时功能非常强大，但它们确实有可能导致一个罕见且难以捉摸的问题，称为死锁。这通常发生在两个线程都尝试获取另一个线程所拥有的互斥体时。这导致它们都被永久阻塞住。在 Android 上，如果线程对于运行应用程序是必不可少的（例如，如果其中一个线程是主线程），它很可能会导致 ANR（应用程序无响应）和应用程序崩溃。

死锁是出了名的不得不处理地令人讨厌的问题。造成这种情况的主要原因是非常低的复现率以及缺乏详细记录问题的日志。后一个问题是由于没有抛出错误/异常的事实造成的。唯一的症状是缺乏输出，以及应用程序没有响应任何消息（例如按键，意图广播等），这反过来又导致 Android 系统通过对话框提示用户，询问他或她是否希望杀死应用程序（臭名昭著的ANR）。

### 死锁的类比
在尝试说明死锁时，我们在现实生活的类比显得有点捉襟见肘，但让我们试一试吧。假设小明和小红负责管理对两个共享资源的访问，比如日志文件和数据库连接。小明和小红约定，任何想使用数据库的人都必须持有特定的红色标记，而任何想要访问日志文件的人都必须持有特定的黑色标记。

这很好地运行了一段时间，直到某一刻发生了以下情况：

1. 小明抓住红色标记并开始写入数据库。
2. 小红抓住黑色标记并开始写一些日志。
3. 在仍然保持红色标记的同时，小明也尝试获取黑色标记以写入关于数据库活动的日志。
4. 由于黑色标记不可用，小明停下来等待小红完成它。
5. 在仍然拿着黑色标记的同时，小红也尝试获取红色标记以检查一些数据库活动。
6. 由于红色标记不可用，小红停下来并等待小明完成它。

正如你所看到的，小红和小明现在都拥有其中一个标记，并等待另一个人释放他们的标记。现在无法取得任何进展，活动将永久阻塞。我们陷入了僵局。

### 解决死锁
如果在你的代码当中检测到了这种情况，解决这个问题的方法是改变获取锁的顺序。这种策略有事被称为建立互斥体层次结构。比方说小明和小红为了获得两个标记，约定必须首先抓住红色标记，然后再抓住黑色标记（而不是反过来）。上面的情景会如此展开：

1. 小明抓住红色标记并开始写入数据库。
2. 小红抓住黑色标记并开始写入一些日志。
3. 在仍然保持红色标记的提示，小明也尝试获取黑色标记以写入关于数据库活动的日志。
4. 由于黑色标记不可用，小明停下来等到小红完成它。
5. 小红需要两个标记，但由于上述协议，她首先释放黑色标记。
6. 小红现在尝试获取红色标记，但是由于它不可用，她停下来等待鲍勃完成它。
7. 小明抓住黑色标记（现在可用）并执行他的工作。
8. 小明释放黑色标记
9. 小明释放红色标记。
10. 小红获得现在可用的红色标记。
11. 小红获得黑色标记。
12. 小红执行她的工作。
13. 小红释放黑色标记。
14. 小红释放红色标记。

因此，这种规则没有可能出现死锁。


## 处理 ANR
如果你的应用程序不幸以 ANR 结束，那么你将不得不使用的唯一东西就是 stats 和 ANR 跟踪。你可能无法重现它，而且你从用户那里得到的反馈在最好的情况是毫无用处，最坏的情况是产生误导。

首先，请注意，如果你使用某些第三方报告崩溃的方式（如HockeyApp），你的ANR将不会显示在那里。你必须转到 Google Play 控制台，点击 Crashes & ANRs，并确保将切换按钮设置为 ANR。然后选择表格中显示的问题之一，现在你会看到统计信息和 ANR 跟踪。

首先看看统计数据。这些可以告诉你哪些 Android 版本和手机受到影响，以及问题何时开始（通常是其中一个应用版本）。

然后你必须查看 ANR traces。这些一开始看起来很吓人，但只要耐心一点，你就可以理解它们。

### 理解 ANR traces
ANR trace 与你在各种异常（例如 <code>NullPointerException</code>）中获得的更为熟悉的堆栈跟踪非常相似，但有一些显著的差异：

- 它在顶部有一个非常大的标题，横跨几行，包含有关该进程的信息。如果你不理解其中的大部分，没有关系，很少有 android 应用程序开发者能做到，而且那里很少有 ANR 的解决方案。
- 在那里有不止一个堆栈跟踪 - 事实上可以有很多。原因是，与更常见的你的代码在其中一个应用程序线程中抛出异常的情况不同，如果是 ANR，应用程序已停止运行，Android 可以执行的最佳操作是转储所有线程的状态，因为没有办法知道哪一个是有问题的。
- 由于没有异常抛出，因此没有异常名称存在。
- 每个线程的顶部都有一个线程头文件，列出了它的名称和状态：
```
"ReferenceQueueDaemon" daemon prio=5 tid=3 Waiting
```
- 很多堆栈跟踪条目是隐藏的，因为它们显示的是 <code>native</code> 的代码：
```
native: #00 pc 000000000048df54  /system/lib64/libart.so (_ZN3art15DumpNativeStackERNSt3__113basic_ostreamIcNS0_11char_traitsIcEEEEiPKcPNS_9ArtMethodEPv+236)  
```

一般来说，结构是这样的：
```
----- pid 3775 at 2017-03-14 11:28:21 -----
Cmd line: ....  
Build fingerprint: ....  
ABI: 'x86'  
....

DALVIK THREADS (16):  
"thread name" ... Status
  | group="main" ...
  | held mutexes= ...
  at java.lang.Thread.sleep!(Native method)
  - sleeping on <0x0c075403> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:1031)
  - locked <0x0c075403> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:985)
  at ...
  at ...
...

"thread name2" ... Status
  | group= ...
  | held mutexes= ...
  at java.lang.Thread.sleep!(Native method)
  - sleeping on <0x0c075403> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:1031)
  - locked <0x0c075403> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:985)
  at ...
  at ...

...
```

通常，你需要的信息位于线程条目里，所以追踪 ANR 时尽量关注这些部分。特别注意你的主线程（tid = 1），它是处理 UI 操作的线程。ANR 始终是由于此线程被阻塞或执行太多工作而导致的。

### 检测死锁
trace 中的每个线程条目包含一些可以帮助你检测死锁的提示。首先你可以看到线程的状态：它通常是 <code>Runnable</code>，<code>Sleeping</code>，<code>Waiting</code>，<code>Native</code> 或 <code>Blocked</code>。这告诉你 ANR 发生时线程处于什么状态，这里特别注意那些被标记为 <code>Blocked</code> 状态的线程。

标记为阻塞的线程通常会告诉你他们正在尝试获取的互斥锁（锁）以及持有该锁的线程的线程标识（tid）。然后，你可以向下滚动到列表中与该线程对应的条目，如果你发现该线程被阻塞，则可以查看它尝试获取的互斥锁以及哪个线程正在锁定这个互斥锁。通常这是第一个线程，这意味着你已经检测到了死锁。

让我们来看一个真实的例子（注意：这个 trace 是真实的，但经过了修改，删除所有可识别的信息）：
```
----- pid 25967 at 2017-03-30 05:32:57 -----  
Cmd line: com.example  
Build fingerprint:   
ABI: 'arm64'  
Build type: optimized  
Zygote loaded classes=4071 post zygote classes=1552  
Intern table: 57141 strong; 443 weak  
JNI: CheckJNI is off; globals=325 (plus 363 weak)  
Libraries: /data/app/com.google.android.gms-1/lib/arm64/libconscrypt_gmscore_jni.so /data/app/com.google.android.gms-1/lib/arm64/libgmscore.so /system/lib64/libandroid.so /system/lib64/libcompiler_rt.so /system/lib64/libfm_jni.so /system/lib64/libhwdeviceinfo.so /system/lib64/libhwtheme_jni.so /system/lib64/libjavacrypto.so /system/lib64/libjnigraphics.so /system/lib64/libmedia_jni.so /system/lib64/libwebviewchromium_loader.so libjavacore.so (12)  
Heap: 20% free, 22MB/28MB; 55958 objects  
Dumping cumulative Gc timings  
Start Dumping histograms for 1 iterations for partial concurrent mark sweep  
ProcessMarkStack:    Sum: 11.069ms 99% C.I. 0.105ms-10.838ms Avg: 3.689ms Max: 10.839ms  
MarkRootsCheckpoint:    Sum: 5.943ms 99% C.I. 2.735ms-3.208ms Avg: 2.971ms Max: 3.208ms  
UpdateAndMarkZygoteModUnionTable:    Sum: 4.417ms 99% C.I. 4.417ms-4.417ms Avg: 4.417ms Max: 4.417ms  
UpdateAndMarkImageModUnionTable:    Sum: 4.200ms 99% C.I. 4.200ms-4.200ms Avg: 4.200ms Max: 4.200ms  
MarkConcurrentRoots:    Sum: 3.501ms 99% C.I. 0.203ms-3.269ms Avg: 1.750ms Max: 3.298ms  
SweepMallocSpace:    Sum: 2.069ms 99% C.I. 0.040ms-2.029ms Avg: 1.034ms Max: 2.029ms  
ScanGrayAllocSpaceObjects:    Sum: 1.014ms 99% C.I. 4us-1010us Avg: 507us Max: 1010us  
ReMarkRoots:    Sum: 830us 99% C.I. 830us-830us Avg: 830us Max: 830us  
MarkAllocStackAsLive:    Sum: 527us 99% C.I. 527us-527us Avg: 527us Max: 527us  
(Paused)ScanGrayAllocSpaceObjects:    Sum: 497us 99% C.I. 0.500us-495.500us Avg: 248.500us Max: 497us
SweepLargeObjects:    Sum: 216us 99% C.I. 216us-216us Avg: 216us Max: 216us  
ImageModUnionClearCards:    Sum: 167us 99% C.I. 57us-110us Avg: 83.500us Max: 110us  
SweepSystemWeaks:    Sum: 93us 99% C.I. 93us-93us Avg: 93us Max: 93us  
AllocSpaceClearCards:    Sum: 84us 99% C.I. 1us-55us Avg: 21us Max: 55us  
FinishPhase:    Sum: 81us 99% C.I. 81us-81us Avg: 81us Max: 81us  
EnqueueFinalizerReferences:    Sum: 78us 99% C.I. 78us-78us Avg: 78us Max: 78us  
MarkNonThreadRoots:    Sum: 69us 99% C.I. 31us-38us Avg: 34.500us Max: 38us  
ScanGrayImageSpaceObjects:    Sum: 56us 99% C.I. 56us-56us Avg: 56us Max: 56us  
(Paused)ScanGrayImageSpaceObjects:    Sum: 53us 99% C.I. 53us-53us Avg: 53us Max: 53us
ZygoteModUnionClearCards:    Sum: 35us 99% C.I. 15us-20us Avg: 17.500us Max: 20us  
RevokeAllThreadLocalAllocationStacks:    Sum: 24us 99% C.I. 24us-24us Avg: 24us Max: 24us  
(Paused)PausePhase:    Sum: 20us 99% C.I. 20us-20us Avg: 20us Max: 20us
(Paused)ProcessMarkStack:    Sum: 18us 99% C.I. 18us-18us Avg: 18us Max: 18us
MarkingPhase:    Sum: 17us 99% C.I. 17us-17us Avg: 17us Max: 17us  
ScanGrayZygoteSpaceObjects:    Sum: 15us 99% C.I. 15us-15us Avg: 15us Max: 15us  
ProcessReferences:    Sum: 11us 99% C.I. 11us-11us Avg: 11us Max: 11us  
PreCleanCards:    Sum: 10us 99% C.I. 10us-10us Avg: 10us Max: 10us  
(Paused)ScanGrayZygoteSpaceObjects:    Sum: 9us 99% C.I. 9us-9us Avg: 9us Max: 9us
ProcessCards:    Sum: 8us 99% C.I. 4us-4us Avg: 4us Max: 4us  
MarkRoots:    Sum: 6us 99% C.I. 6us-6us Avg: 6us Max: 6us  
BindBitmaps:    Sum: 4us 99% C.I. 4us-4us Avg: 4us Max: 4us  
RecursiveMark:    Sum: 2us 99% C.I. 2us-2us Avg: 2us Max: 2us  
SweepZygoteSpace:    Sum: 1us 99% C.I. 1us-1us Avg: 1us Max: 1us  
FindDefaultSpaceBitmap:    Sum: 0 99% C.I. 0ns-0ns Avg: 0ns Max: 0ns  
Done Dumping histograms  
partial concurrent mark sweep paused:    Sum: 3.368ms 99% C.I. 3.368ms-3.368ms Avg: 3.368ms Max: 3.368ms  
partial concurrent mark sweep total time: 35.187ms mean time: 35.187ms  
partial concurrent mark sweep freed: 9904 objects with total size 1088KB  
partial concurrent mark sweep throughput: 282971/s / 30MB/s  
Start Dumping histograms for 1 iterations for sticky concurrent mark sweep  
FreeList:    Sum: 6.592ms 99% C.I. 9us-448.750us Avg: 160.780us Max: 457us  
SweepSystemWeaks:    Sum: 4.712ms 99% C.I. 4.712ms-4.712ms Avg: 4.712ms Max: 4.712ms  
ProcessMarkStack:    Sum: 3.085ms 99% C.I. 0.333us-2990us Avg: 771.250us Max: 3038us  
MarkConcurrentRoots:    Sum: 2.759ms 99% C.I. 0.017ms-2.723ms Avg: 1.379ms Max: 2.742ms  
MarkRootsCheckpoint:    Sum: 2.357ms 99% C.I. 0.699ms-1.658ms Avg: 1.178ms Max: 1.658ms  
SweepArray:    Sum: 2.075ms 99% C.I. 2.075ms-2.075ms Avg: 2.075ms Max: 2.075ms  
ScanGrayAllocSpaceObjects:    Sum: 1.745ms 99% C.I. 0.500us-934us Avg: 436.250us Max: 934us  
ScanGrayImageSpaceObjects:    Sum: 1.419ms 99% C.I. 33us-1386us Avg: 709.500us Max: 1386us  
ScanGrayZygoteSpaceObjects:    Sum: 362us 99% C.I. 6us-356us Avg: 181us Max: 356us  
ResetStack:    Sum: 204us 99% C.I. 204us-204us Avg: 204us Max: 204us  
MarkingPhase:    Sum: 200us 99% C.I. 200us-200us Avg: 200us Max: 200us  
ReMarkRoots:    Sum: 157us 99% C.I. 157us-157us Avg: 157us Max: 157us  
AllocSpaceClearCards:    Sum: 123us 99% C.I. 0.500us-67us Avg: 30.750us Max: 67us  
(Paused)ScanGrayAllocSpaceObjects:    Sum: 68us 99% C.I. 0.500us-68us Avg: 34us Max: 68us
MarkNonThreadRoots:    Sum: 59us 99% C.I. 24us-35us Avg: 29.500us Max: 35us  
ZygoteModUnionClearCards:    Sum: 46us 99% C.I. 11us-35us Avg: 23us Max: 35us  
FinishPhase:    Sum: 38us 99% C.I. 38us-38us Avg: 38us Max: 38us  
(Paused)ScanGrayImageSpaceObjects:    Sum: 31us 99% C.I. 31us-31us Avg: 31us Max: 31us
(Paused)PausePhase:    Sum: 30us 99% C.I. 30us-30us Avg: 30us Max: 30us
EnqueueFinalizerReferences:    Sum: 29us 99% C.I. 29us-29us Avg: 29us Max: 29us  
ReclaimPhase:    Sum: 17us 99% C.I. 17us-17us Avg: 17us Max: 17us  
InitializePhase:    Sum: 14us 99% C.I. 14us-14us Avg: 14us Max: 14us  
ProcessCards:    Sum: 11us 99% C.I. 4us-7us Avg: 5.500us Max: 7us  
PreCleanCards:    Sum: 7us 99% C.I. 7us-7us Avg: 7us Max: 7us  
(Paused)ScanGrayZygoteSpaceObjects:    Sum: 6us 99% C.I. 6us-6us Avg: 6us Max: 6us
ProcessReferences:    Sum: 5us 99% C.I. 5us-5us Avg: 5us Max: 5us  
MarkRoots:    Sum: 4us 99% C.I. 4us-4us Avg: 4us Max: 4us  
UnBindBitmaps:    Sum: 3us 99% C.I. 3us-3us Avg: 3us Max: 3us  
RecordFree:    Sum: 2us 99% C.I. 2us-2us Avg: 2us Max: 2us  
ForwardSoftReferences:    Sum: 1us 99% C.I. 1us-1us Avg: 1us Max: 1us  
(Paused)ProcessMarkStack:    Sum: 0 99% C.I. 0ns-0ns Avg: 0ns Max: 0ns
Done Dumping histograms  
sticky concurrent mark sweep paused:    Sum: 1.669ms 99% C.I. 1.669ms-1.669ms Avg: 1.669ms Max: 1.669ms  
sticky concurrent mark sweep total time: 26.304ms mean time: 26.304ms  
sticky concurrent mark sweep freed: 40059 objects with total size 2MB  
sticky concurrent mark sweep throughput: 1.54073e+06/s / 112MB/s  
Start Dumping histograms for 2 iterations for marksweep + semispace  
MarkRoots:    Sum: 76.622ms 99% C.I. 31.604ms-45.018ms Avg: 38.311ms Max: 45.018ms  
ProcessMarkStack:    Sum: 51.438ms 99% C.I. 0.008ms-30.576ms Avg: 12.859ms Max: 30.576ms  
ClearCardTable:    Sum: 12.596ms 99% C.I. 2.902ms-9.633ms Avg: 6.298ms Max: 9.694ms  
UpdateAndMarkImageModUnionTable:    Sum: 5.851ms 99% C.I. 1.762ms-4.077ms Avg: 2.925ms Max: 4.089ms  
MarkStackAsLive:    Sum: 2.255ms 99% C.I. 0.104ms-2.151ms Avg: 1.127ms Max: 2.151ms  
UpdateAndMarkZygoteModUnionTable:    Sum: 992us 99% C.I. 477us-515us Avg: 496us Max: 515us  
(Paused)EnqueueFinalizerReferences:    Sum: 819us 99% C.I. 51us-768us Avg: 409.500us Max: 768us
SweepSystemWeaks:    Sum: 570us 99% C.I. 263us-307us Avg: 285us Max: 307us  
RevokeAllThreadLocalBuffers:    Sum: 519us 99% C.I. 65us-193us Avg: 129.750us Max: 193us  
SweepLargeObjects:    Sum: 353us 99% C.I. 25us-328us Avg: 176.500us Max: 328us  
ImageModUnionClearCards:    Sum: 204us 99% C.I. 82us-122us Avg: 102us Max: 122us  
FinishPhase:    Sum: 174us 99% C.I. 58us-116us Avg: 87us Max: 116us  
SweepAllocSpace:    Sum: 105us 99% C.I. 35us-70us Avg: 52.500us Max: 70us  
(Paused)ProcessReferences:    Sum: 43us 99% C.I. 17us-26us Avg: 21.500us Max: 26us
SwapBitmaps:    Sum: 39us 99% C.I. 16us-23us Avg: 19.500us Max: 23us  
MarkReachableObjects:    Sum: 32us 99% C.I. 14us-18us Avg: 16us Max: 18us  
BindBitmaps:    Sum: 20us 99% C.I. 7us-13us Avg: 10us Max: 13us  
ProcessCards:    Sum: 16us 99% C.I. 5us-11us Avg: 8us Max: 11us  
Sweep:    Sum: 14us 99% C.I. 6us-8us Avg: 7us Max: 8us  
MarkingPhase:    Sum: 10us 99% C.I. 2us-8us Avg: 5us Max: 8us  
InitializePhase:    Sum: 9us 99% C.I. 2us-7us Avg: 4.500us Max: 7us  
ReclaimPhase:    Sum: 7us 99% C.I. 3us-4us Avg: 3.500us Max: 4us  
PreSweepingGcVerification:    Sum: 3us 99% C.I. 1us-2us Avg: 1.500us Max: 2us  
SweepZygoteSpace:    Sum: 2us 99% C.I. 1us-1us Avg: 1us Max: 1us  
PreGcVerificationPaused:    Sum: 1us 99% C.I. 250ns-1000ns Avg: 500ns Max: 1000ns  
PostGcVerificationPaused:    Sum: 0 99% C.I. 0ns-0ns Avg: 0ns Max: 0ns  
Done Dumping histograms  
marksweep + semispace paused:    Sum: 153.066ms 99% C.I. 71.381ms-81.685ms Avg: 76.533ms Max: 81.685ms  
marksweep + semispace total time: 152.772ms mean time: 76.386ms  
marksweep + semispace freed: 85295 objects with total size 4MB  
marksweep + semispace throughput: 561151/s / 32MB/s  
Total time spent in GC: 214.263ms  
Mean GC size throughput: 15MB/s  
Mean GC object throughput: 233017 objects/s  
Total number of allocations 105885  
Total bytes allocated 26MB  
Total bytes freed 3MB  
Free memory 5MB  
Free memory until GC 5MB  
Free memory until OOME 233MB  
Total memory 28MB  
Max memory 256MB  
Zygote space size 1540KB  
Total mutator paused time: 158.103ms  
Total time waiting for GC to complete: 5.416us  
Total GC count: 4  
Total GC time: 214.263ms  
Total blocking GC count: 0  
Total blocking GC time: 0  
Histogram of GC count per 10000 ms: 0:10,1:1,2:1  
Histogram of blocking GC count per 10000 ms: 0:12

suspend all histogram:    Sum: 3.470ms 99% C.I. 1us-1882.560us Avg: 266.923us Max: 1896us  
DALVIK THREADS (21):  
"Signal Catcher" daemon prio=5 tid=2 Runnable
  | group="system" sCount=0 dsCount=0 obj=0x12c0d0a0 self=0x55999df770
  | sysTid=25972 nice=0 cgrp=top_visible sched=0/0 handle=0x7f78de7450
  | state=R schedstat=( 6967792 0 10 ) utm=0 stm=0 core=3 HZ=100
  | stack=0x7f78ceb000-0x7f78ced000 stackSize=1013KB
  | held mutexes= "mutator lock"(shared held)
  native: #00 pc 000000000048df54  /system/lib64/libart.so (_ZN3art15DumpNativeStackERNSt3__113basic_ostreamIcNS0_11char_traitsIcEEEEiPKcPNS_9ArtMethodEPv+236)
  native: #01 pc 000000000045d404  /system/lib64/libart.so (_ZNK3art6Thread4DumpERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+220)
  native: #02 pc 0000000000469c70  /system/lib64/libart.so (_ZN3art14DumpCheckpoint3RunEPNS_6ThreadE+688)
  native: #03 pc 000000000046ab8c  /system/lib64/libart.so (_ZN3art10ThreadList13RunCheckpointEPNS_7ClosureE+276)
  native: #04 pc 000000000046b248  /system/lib64/libart.so (_ZN3art10ThreadList4DumpERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+188)
  native: #05 pc 000000000046bb2c  /system/lib64/libart.so (_ZN3art10ThreadList14DumpForSigQuitERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+492)
  native: #06 pc 0000000000435064  /system/lib64/libart.so (_ZN3art7Runtime14DumpForSigQuitERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+96)
  native: #07 pc 000000000044281c  /system/lib64/libart.so (_ZN3art13SignalCatcher13HandleSigQuitEv+1256)
  native: #08 pc 000000000044342c  /system/lib64/libart.so (_ZN3art13SignalCatcher3RunEPv+452)
  native: #09 pc 0000000000068464  /system/lib64/libc.so (_ZL15__pthread_startPv+52)
  native: #10 pc 000000000001d544  /system/lib64/libc.so (__start_thread+16)
  (no managed stack frames)

"main" prio=5 tid=1 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x7477dea0 self=0x5597d71b70
  | sysTid=25967 nice=-1 cgrp=top_visible sched=0/0 handle=0x7f7dcb5000
  | state=S schedstat=( 688872496 20759648 497 ) utm=57 stm=11 core=0 HZ=100
  | stack=0x7fc4bc9000-0x7fc4bcb000 stackSize=8MB
  | held mutexes=
  at com.google.android.gms.tagmanager.zzp.zzav(unavailable:-1)
  - waiting to lock <0x08362446> (a com.google.android.gms.tagmanager.zzp) held by thread 15
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  at com.google.android.gms.tagmanager.zzp$zzd.zzOE(unavailable:-1)
  at com.google.android.gms.tagmanager.zzo.refresh(unavailable:-1)
  - locked <0x023ef607> (a com.google.android.gms.tagmanager.zzo)
  at com.example.MyApplication$1.onSuccess(MyApplication.java:176)
  at com.example.MyApplication$1.onSuccess(MyApplication.java:169)
  at com.example.callback.GenericCallBack.handleSuccess(GenericCallBack.java:39)
  at com.example.service.AnalyticsServiceImpl$1.onResult(AnalyticsServiceImpl.java:74)
  at com.example.service.AnalyticsServiceImpl$1.onResult(AnalyticsServiceImpl.java:65)
  at com.google.android.gms.internal.zzzx$zza.zzb(unavailable:-1)
  at com.google.android.gms.internal.zzzx$zza.handleMessage(unavailable:-1)
  at android.os.Handler.dispatchMessage(Handler.java:102)
  at android.os.Looper.loop(Looper.java:150)
  at android.app.ActivityThread.main(ActivityThread.java:5546)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:794)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:684)

"ReferenceQueueDaemon" daemon prio=5 tid=3 Waiting
  | group="system" sCount=1 dsCount=0 obj=0x12c0d100 self=0x559992ff30
  | sysTid=25973 nice=0 cgrp=top_visible sched=0/0 handle=0x7f78cdf450
  | state=S schedstat=( 1998256 1180608 19 ) utm=0 stm=0 core=0 HZ=100
  | stack=0x7f78bdd000-0x7f78bdf000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x042f1334> (a java.lang.Class)
  at java.lang.Daemons$ReferenceQueueDaemon.run(Daemons.java:147)
  - locked <0x042f1334> (a java.lang.Class)
  at java.lang.Thread.run(Thread.java:833)

"FinalizerDaemon" daemon prio=5 tid=4 Waiting
  | group="system" sCount=1 dsCount=0 obj=0x12c0d160 self=0x5597d6e0d0
  | sysTid=25974 nice=0 cgrp=top_visible sched=0/0 handle=0x7f78bd3450
  | state=S schedstat=( 9917648 1193088 41 ) utm=0 stm=0 core=1 HZ=100
  | stack=0x7f78ad1000-0x7f78ad3000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x0e1d355d> (a java.lang.ref.ReferenceQueue)
  at java.lang.Object.wait(Object.java:423)
  at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:101)
  - locked <0x0e1d355d> (a java.lang.ref.ReferenceQueue)
  at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:72)
  at java.lang.Daemons$FinalizerDaemon.run(Daemons.java:185)
  at java.lang.Thread.run(Thread.java:833)

"HeapTaskDaemon" daemon prio=5 tid=5 Blocked
  | group="system" sCount=1 dsCount=0 obj=0x12c0d1c0 self=0x55995bd630
  | sysTid=25976 nice=0 cgrp=top_visible sched=0/0 handle=0x7f789c0450
  | state=S schedstat=( 209432704 2489136 42 ) utm=19 stm=1 core=4 HZ=100
  | stack=0x7f788be000-0x7f788c0000 stackSize=1037KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/25976/stack)
  native: #00 pc 000000000001a8d4  /system/lib64/libc.so (syscall+28)
  native: #01 pc 000000000013a02c  /system/lib64/libart.so (_ZN3art17ConditionVariable4WaitEPNS_6ThreadE+136)
  native: #02 pc 000000000026a1f4  /system/lib64/libart.so (_ZN3art2gc13TaskProcessor7GetTaskEPNS_6ThreadE+128)
  native: #03 pc 000000000026a85c  /system/lib64/libart.so (_ZN3art2gc13TaskProcessor11RunAllTasksEPNS_6ThreadE+120)
  native: #04 pc 000000000000054c  /data/dalvik-cache/arm64/system@framework@boot.oat (Java_dalvik_system_VMRuntime_runHeapTasks__+128)
  at dalvik.system.VMRuntime.runHeapTasks(Native method)
  - waiting to lock an unknown object
  at java.lang.Daemons$HeapTaskDaemon.run(Daemons.java:355)
  at java.lang.Thread.run(Thread.java:833)

"FinalizerWatchdogDaemon" daemon prio=5 tid=6 Waiting
  | group="system" sCount=1 dsCount=0 obj=0x12c0d220 self=0x559855a550
  | sysTid=25975 nice=0 cgrp=top_visible sched=0/0 handle=0x7f78acc450
  | state=S schedstat=( 1235520 0 10 ) utm=0 stm=0 core=2 HZ=100
  | stack=0x7f789ca000-0x7f789cc000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x0ba03ed2> (a java.lang.Daemons$FinalizerWatchdogDaemon)
  at java.lang.Daemons$FinalizerWatchdogDaemon.waitForObject(Daemons.java:255)
  - locked <0x0ba03ed2> (a java.lang.Daemons$FinalizerWatchdogDaemon)
  at java.lang.Daemons$FinalizerWatchdogDaemon.run(Daemons.java:227)
  at java.lang.Thread.run(Thread.java:833)

"Binder_1" prio=5 tid=7 Native
  | group="main" sCount=1 dsCount=0 obj=0x12c0d280 self=0x5599938310
  | sysTid=25977 nice=0 cgrp=top_visible sched=0/0 handle=0x7f63b1e450
  | state=S schedstat=( 6009328 2418208 23 ) utm=0 stm=0 core=3 HZ=100
  | stack=0x7f63a22000-0x7f63a24000 stackSize=1013KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/25977/stack)
  native: #00 pc 000000000006af08  /system/lib64/libc.so (__ioctl+4)
  native: #01 pc 0000000000074a00  /system/lib64/libc.so (ioctl+100)
  native: #02 pc 000000000002d494  /system/lib64/libbinder.so (_ZN7android14IPCThreadState14talkWithDriverEb+164)
  native: #03 pc 000000000002dce8  /system/lib64/libbinder.so (_ZN7android14IPCThreadState20getAndExecuteCommandEv+24)
  native: #04 pc 000000000002de04  /system/lib64/libbinder.so (_ZN7android14IPCThreadState14joinThreadPoolEb+76)
  native: #05 pc 0000000000036858  /system/lib64/libbinder.so (???)
  native: #06 pc 0000000000016394  /system/lib64/libutils.so (_ZN7android6Thread11_threadLoopEPv+208)
  native: #07 pc 0000000000092bf0  /system/lib64/libandroid_runtime.so (_ZN7android14AndroidRuntime15javaThreadShellEPv+96)
  native: #08 pc 0000000000015be4  /system/lib64/libutils.so (???)
  native: #09 pc 0000000000068464  /system/lib64/libc.so (_ZL15__pthread_startPv+52)
  native: #10 pc 000000000001d544  /system/lib64/libc.so (__start_thread+16)
  (no managed stack frames)

"Binder_2" prio=5 tid=8 Native
  | group="main" sCount=1 dsCount=0 obj=0x12c0d2e0 self=0x5599935b10
  | sysTid=25978 nice=0 cgrp=top_visible sched=0/0 handle=0x7f63a1f450
  | state=S schedstat=( 5317520 869440 23 ) utm=0 stm=0 core=1 HZ=100
  | stack=0x7f63923000-0x7f63925000 stackSize=1013KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/25978/stack)
  native: #00 pc 000000000006af08  /system/lib64/libc.so (__ioctl+4)
  native: #01 pc 0000000000074a00  /system/lib64/libc.so (ioctl+100)
  native: #02 pc 000000000002d494  /system/lib64/libbinder.so (_ZN7android14IPCThreadState14talkWithDriverEb+164)
  native: #03 pc 000000000002dce8  /system/lib64/libbinder.so (_ZN7android14IPCThreadState20getAndExecuteCommandEv+24)
  native: #04 pc 000000000002de04  /system/lib64/libbinder.so (_ZN7android14IPCThreadState14joinThreadPoolEb+76)
  native: #05 pc 0000000000036858  /system/lib64/libbinder.so (???)
  native: #06 pc 0000000000016394  /system/lib64/libutils.so (_ZN7android6Thread11_threadLoopEPv+208)
  native: #07 pc 0000000000092bf0  /system/lib64/libandroid_runtime.so (_ZN7android14AndroidRuntime15javaThreadShellEPv+96)
  native: #08 pc 0000000000015be4  /system/lib64/libutils.so (???)
  native: #09 pc 0000000000068464  /system/lib64/libc.so (_ZL15__pthread_startPv+52)
  native: #10 pc 000000000001d544  /system/lib64/libc.so (__start_thread+16)
  (no managed stack frames)

"pool-5-thread-1" prio=5 tid=11 Waiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d340 self=0x5598b73790
  | sysTid=26000 nice=0 cgrp=top_visible sched=0/0 handle=0x7f61cf6450
  | state=S schedstat=( 15990624 1283568 26 ) utm=1 stm=0 core=2 HZ=100
  | stack=0x7f61bf4000-0x7f61bf6000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x0ecaeba3> (a java.lang.Object)
  at java.lang.Thread.parkFor$(Thread.java:1235)
  - locked <0x0ecaeba3> (a java.lang.Object)
  at sun.misc.Unsafe.park(Unsafe.java:299)
  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:158)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2013)
  at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:410)
  at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1036)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1098)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

"pool-6-thread-1" prio=5 tid=12 Waiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d3a0 self=0x55999908b0
  | sysTid=26001 nice=0 cgrp=top_visible sched=0/0 handle=0x7f61bf1450
  | state=S schedstat=( 13138944 1240720 14 ) utm=1 stm=0 core=6 HZ=100
  | stack=0x7f61aef000-0x7f61af1000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x03442ea0> (a java.lang.Object)
  at java.lang.Thread.parkFor$(Thread.java:1235)
  - locked <0x03442ea0> (a java.lang.Object)
  at sun.misc.Unsafe.park(Unsafe.java:299)
  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:158)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2013)
  at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:410)
  at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1036)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1098)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

"Thread-1658" prio=5 tid=13 TimedWaiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d400 self=0x5598ca44c0
  | sysTid=26002 nice=10 cgrp=top_visible sched=0/0 handle=0x7f61aec450
  | state=S schedstat=( 18796336 3701568 40 ) utm=1 stm=0 core=2 HZ=100
  | stack=0x7f619ea000-0x7f619ec000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x09fcca59> (a java.lang.Object)
  at java.lang.Object.wait(Object.java:423)
  at com.google.android.gms.tagmanager.zza.zzOu(unavailable:-1)
  - locked <0x09fcca59> (a java.lang.Object)
  at com.google.android.gms.tagmanager.zza.zzb(unavailable:-1)
  at com.google.android.gms.tagmanager.zza$2.run(unavailable:-1)
  at java.lang.Thread.run(Thread.java:833)

"pool-7-thread-1" prio=5 tid=14 Waiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d460 self=0x55985c4f50
  | sysTid=26005 nice=0 cgrp=top_visible sched=0/0 handle=0x7f6174b450
  | state=S schedstat=( 12652016 149760 18 ) utm=1 stm=0 core=3 HZ=100
  | stack=0x7f61649000-0x7f6164b000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x001f761e> (a java.lang.Object)
  at java.lang.Thread.parkFor$(Thread.java:1235)
  - locked <0x001f761e> (a java.lang.Object)
  at sun.misc.Unsafe.park(Unsafe.java:299)
  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:158)
  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2013)
  at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:410)
  at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1036)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1098)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

"pool-8-thread-1" prio=5 tid=15 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x12c0d4c0 self=0x5598cd6440
  | sysTid=26006 nice=0 cgrp=top_visible sched=0/0 handle=0x7f61646450
  | state=S schedstat=( 64174864 5461664 62 ) utm=6 stm=0 core=3 HZ=100
  | stack=0x7f61544000-0x7f61546000 stackSize=1037KB
  | held mutexes=
  at com.google.android.gms.tagmanager.zzo.zza(unavailable:-1)
  - waiting to lock <0x023ef607> (a com.google.android.gms.tagmanager.zzo) held by thread 1
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  - locked <0x08362446> (a com.google.android.gms.tagmanager.zzp)
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  at com.google.android.gms.tagmanager.zzp$zzc.zzb(unavailable:-1)
  - locked <0x08362446> (a com.google.android.gms.tagmanager.zzp)
  at com.google.android.gms.tagmanager.zzp$zzc.onSuccess(unavailable:-1)
  at com.google.android.gms.tagmanager.zzct.zzPD(unavailable:-1)
  at com.google.android.gms.tagmanager.zzct.run(unavailable:-1)
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:423)
  at java.util.concurrent.FutureTask.run(FutureTask.java:237)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:154)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:269)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

"WonderPush" prio=5 tid=16 Native
  | group="main" sCount=1 dsCount=0 obj=0x12c0d520 self=0x5598c83520
  | sysTid=26007 nice=0 cgrp=top_visible sched=0/0 handle=0x7f61541450
  | state=S schedstat=( 86143824 20670208 187 ) utm=8 stm=0 core=0 HZ=100
  | stack=0x7f6143f000-0x7f61441000 stackSize=1037KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/26007/stack)
  native: #00 pc 000000000006b674  /system/lib64/libc.so (__epoll_pwait+8)
  native: #01 pc 000000000001dba4  /system/lib64/libc.so (epoll_pwait+32)
  native: #02 pc 000000000001b96c  /system/lib64/libutils.so (_ZN7android6Looper9pollInnerEi+144)
  native: #03 pc 000000000001bd4c  /system/lib64/libutils.so (_ZN7android6Looper8pollOnceEiPiS1_PPv+80)
  native: #04 pc 00000000000d7e2c  /system/lib64/libandroid_runtime.so (_ZN7android18NativeMessageQueue8pollOnceEP7_JNIEnvP8_jobjecti+48)
  native: #05 pc 000000000000082c  /data/dalvik-cache/arm64/system@framework@boot.oat (Java_android_os_MessageQueue_nativePollOnce__JI+144)
  at android.os.MessageQueue.nativePollOnce(Native method)
  at android.os.MessageQueue.next(MessageQueue.java:330)
  at android.os.Looper.loop(Looper.java:137)
  at com.wonderpush.sdk.WonderPush$1.run(WonderPush.java:67)
  at java.lang.Thread.run(Thread.java:833)

"Thread-1663" prio=5 tid=17 Sleeping
  | group="main" sCount=1 dsCount=0 obj=0x12c0d580 self=0x5598c95130
  | sysTid=26009 nice=0 cgrp=top_visible sched=0/0 handle=0x7f6143c450
  | state=S schedstat=( 2586896 703248 4 ) utm=0 stm=0 core=2 HZ=100
  | stack=0x7f6133a000-0x7f6133c000 stackSize=1037KB
  | held mutexes=
  at java.lang.Thread.sleep!(Native method)
  - sleeping on <0x0c969eff> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:1046)
  - locked <0x0c969eff> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:1000)
  at com.wonderpush.sdk.WonderPushRequestVault$1.run(WonderPushRequestVault.java:78)
  at java.lang.Thread.run(Thread.java:833)

"Okio Watchdog" daemon prio=5 tid=18 Waiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d5e0 self=0x55999f4a50
  | sysTid=26014 nice=0 cgrp=top_visible sched=0/0 handle=0x7f59452450
  | state=S schedstat=( 590928 0 4 ) utm=0 stm=0 core=3 HZ=100
  | stack=0x7f59350000-0x7f59352000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x08cd94cc> (a java.lang.Class)
  at com.android.okhttp.okio.AsyncTimeout.awaitTimeout(AsyncTimeout.java:311)
  - locked <0x08cd94cc> (a java.lang.Class)
  at com.android.okhttp.okio.AsyncTimeout.access$000(AsyncTimeout.java:40)
  at com.android.okhttp.okio.AsyncTimeout$Watchdog.run(AsyncTimeout.java:286)

"OkHttp ConnectionPool" daemon prio=5 tid=19 TimedWaiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d640 self=0x5599b2edb0
  | sysTid=26016 nice=0 cgrp=top_visible sched=0/0 handle=0x7f5934d450
  | state=S schedstat=( 455936 0 3 ) utm=0 stm=0 core=1 HZ=100
  | stack=0x7f5924b000-0x7f5924d000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x04a6b315> (a com.android.okhttp.ConnectionPool)
  at com.android.okhttp.ConnectionPool.performCleanup(ConnectionPool.java:305)
  - locked <0x04a6b315> (a com.android.okhttp.ConnectionPool)
  at com.android.okhttp.ConnectionPool.runCleanupUntilPoolIsEmpty(ConnectionPool.java:242)
  at com.android.okhttp.ConnectionPool.access$000(ConnectionPool.java:54)
  at com.android.okhttp.ConnectionPool$1.run(ConnectionPool.java:97)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

"RenderThread" prio=5 tid=20 Native
  | group="main" sCount=1 dsCount=0 obj=0x12c0d6a0 self=0x5599ba3900
  | sysTid=26017 nice=-4 cgrp=top_visible sched=0/0 handle=0x7f59248450
  | state=S schedstat=( 2012608 2358720 12 ) utm=0 stm=0 core=3 HZ=100
  | stack=0x7f5914c000-0x7f5914e000 stackSize=1013KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/26017/stack)
  native: #00 pc 000000000006b674  /system/lib64/libc.so (__epoll_pwait+8)
  native: #01 pc 000000000001dba4  /system/lib64/libc.so (epoll_pwait+32)
  native: #02 pc 000000000001b96c  /system/lib64/libutils.so (_ZN7android6Looper9pollInnerEi+144)
  native: #03 pc 000000000001bd4c  /system/lib64/libutils.so (_ZN7android6Looper8pollOnceEiPiS1_PPv+80)
  native: #04 pc 000000000002c420  /system/lib64/libhwui.so (_ZN7android10uirenderer12renderthread12RenderThread10threadLoopEv+100)
  native: #05 pc 0000000000016394  /system/lib64/libutils.so (_ZN7android6Thread11_threadLoopEPv+208)
  native: #06 pc 0000000000092bf0  /system/lib64/libandroid_runtime.so (_ZN7android14AndroidRuntime15javaThreadShellEPv+96)
  native: #07 pc 0000000000015be4  /system/lib64/libutils.so (???)
  native: #08 pc 0000000000068464  /system/lib64/libc.so (_ZL15__pthread_startPv+52)
  native: #09 pc 000000000001d544  /system/lib64/libc.so (__start_thread+16)
  (no managed stack frames)

"Okio Watchdog" daemon prio=5 tid=21 Waiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d700 self=0x559852e380
  | sysTid=26018 nice=0 cgrp=top_visible sched=0/0 handle=0x7f5823f450
  | state=S schedstat=( 833040 194480 5 ) utm=0 stm=0 core=0 HZ=100
  | stack=0x7f5813d000-0x7f5813f000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x0cc8d62a> (a java.lang.Class)
  at okio.AsyncTimeout.awaitTimeout(AsyncTimeout.java:311)
  - locked <0x0cc8d62a> (a java.lang.Class)
  at okio.AsyncTimeout.access$000(AsyncTimeout.java:40)
  at okio.AsyncTimeout$Watchdog.run(AsyncTimeout.java:286)

"IntentService[WonderPushRegistrationIntentService]" prio=5 tid=22 Native
  | group="main" sCount=1 dsCount=0 obj=0x12c0d760 self=0x5599b3b010
  | sysTid=26019 nice=0 cgrp=top_visible sched=0/0 handle=0x7f5813a450
  | state=S schedstat=( 4619888 974688 10 ) utm=0 stm=0 core=4 HZ=100
  | stack=0x7f58038000-0x7f5803a000 stackSize=1037KB
  | held mutexes=
  kernel: (couldn't read /proc/self/task/26019/stack)
  native: #00 pc 000000000006b674  /system/lib64/libc.so (__epoll_pwait+8)
  native: #01 pc 000000000001dba4  /system/lib64/libc.so (epoll_pwait+32)
  native: #02 pc 000000000001b96c  /system/lib64/libutils.so (_ZN7android6Looper9pollInnerEi+144)
  native: #03 pc 000000000001bd4c  /system/lib64/libutils.so (_ZN7android6Looper8pollOnceEiPiS1_PPv+80)
  native: #04 pc 00000000000d7e2c  /system/lib64/libandroid_runtime.so (_ZN7android18NativeMessageQueue8pollOnceEP7_JNIEnvP8_jobjecti+48)
  native: #05 pc 000000000000082c  /data/dalvik-cache/arm64/system@framework@boot.oat (Java_android_os_MessageQueue_nativePollOnce__JI+144)
  at android.os.MessageQueue.nativePollOnce(Native method)
  at android.os.MessageQueue.next(MessageQueue.java:330)
  at android.os.Looper.loop(Looper.java:137)
  at android.os.HandlerThread.run(HandlerThread.java:61)

"OkHttp ConnectionPool" daemon prio=5 tid=23 TimedWaiting
  | group="main" sCount=1 dsCount=0 obj=0x12c0d7c0 self=0x5599bbd9e0
  | sysTid=26021 nice=0 cgrp=top_visible sched=0/0 handle=0x7f58035450
  | state=S schedstat=( 798928 0 2 ) utm=0 stm=0 core=3 HZ=100
  | stack=0x7f57f33000-0x7f57f35000 stackSize=1037KB
  | held mutexes=
  at java.lang.Object.wait!(Native method)
  - waiting on <0x01402c1b> (a com.squareup.okhttp.ConnectionPool)
  at com.squareup.okhttp.ConnectionPool.performCleanup(ConnectionPool.java:305)
  - locked <0x01402c1b> (a com.squareup.okhttp.ConnectionPool)
  at com.squareup.okhttp.ConnectionPool.runCleanupUntilPoolIsEmpty(ConnectionPool.java:242)
  at com.squareup.okhttp.ConnectionPool.access$000(ConnectionPool.java:54)
  at com.squareup.okhttp.ConnectionPool$1.run(ConnectionPool.java:97)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)

----- end 25967 -----
```

这看起来很吓人，不是吗？但是我们现在还不应该放弃，我们 Android 开发人员都是铁打的 :)

我们先看看“主”线程：
```
"main" prio=5 tid=1 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x7477dea0 self=0x5597d71b70
  | sysTid=25967 nice=-1 cgrp=top_visible sched=0/0 handle=0x7f7dcb5000
  | state=S schedstat=( 688872496 20759648 497 ) utm=57 stm=11 core=0 HZ=100
  | stack=0x7fc4bc9000-0x7fc4bcb000 stackSize=8MB
  | held mutexes=
  at com.google.android.gms.tagmanager.zzp.zzav(unavailable:-1)
  - waiting to lock <0x08362446> (a com.google.android.gms.tagmanager.zzp) held by thread 15
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  at com.google.android.gms.tagmanager.zzp$zzd.zzOE(unavailable:-1)
  at com.google.android.gms.tagmanager.zzo.refresh(unavailable:-1)
  - locked <0x023ef607> (a com.google.android.gms.tagmanager.zzo)
  at com.example.MyApplication$1.onSuccess(MyApplication.java:176)
  at com.example.MyApplication$1.onSuccess(MyApplication.java:169)
  at com.example.callback.GenericCallBack.handleSuccess(GenericCallBack.java:39)
  at com.example.service.AnalyticsServiceImpl$1.onResult(AnalyticsServiceImpl.java:74)
  at com.example.service.AnalyticsServiceImpl$1.onResult(AnalyticsServiceImpl.java:65)
  at com.google.android.gms.internal.zzzx$zza.zzb(unavailable:-1)
  at com.google.android.gms.internal.zzzx$zza.handleMessage(unavailable:-1)
  at android.os.Handler.dispatchMessage(Handler.java:102)
  at android.os.Looper.loop(Looper.java:150)
  at android.app.ActivityThread.main(ActivityThread.java:5546)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:794)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:684)
```

第一行告诉我们主线程确实被阻塞了，向下滚动一点，我们可以看到直接原因：Google Tagmanager 库中的一行尝试获取线程15所持有的锁：
```
at com.google.android.gms.tagmanager.zzp.zzav(unavailable:-1)
- waiting to lock <0x08362446> (a com.google.android.gms.tagmanager.zzp) held by thread 15
```

由于 Google Tagmanager 库被混淆了（从所有 zzp 和 zzav 名称可知），所以问题的确切行数和锁的名称是未知的。但该行仍然指向链中的下一个罪魁祸首：ID 为15的线程。

现在让我们在 trace 中搜索线程15：
```
"pool-8-thread-1" prio=5 tid=15 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x12c0d4c0 self=0x5598cd6440
  | sysTid=26006 nice=0 cgrp=top_visible sched=0/0 handle=0x7f61646450
  | state=S schedstat=( 64174864 5461664 62 ) utm=6 stm=0 core=3 HZ=100
  | stack=0x7f61544000-0x7f61546000 stackSize=1037KB
  | held mutexes=
  at com.google.android.gms.tagmanager.zzo.zza(unavailable:-1)
  - waiting to lock <0x023ef607> (a com.google.android.gms.tagmanager.zzo) held by thread 1
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  - locked <0x08362446> (a com.google.android.gms.tagmanager.zzp)
  at com.google.android.gms.tagmanager.zzp.zza(unavailable:-1)
  at com.google.android.gms.tagmanager.zzp$zzc.zzb(unavailable:-1)
  - locked <0x08362446> (a com.google.android.gms.tagmanager.zzp)
  at com.google.android.gms.tagmanager.zzp$zzc.onSuccess(unavailable:-1)
  at com.google.android.gms.tagmanager.zzct.zzPD(unavailable:-1)
  at com.google.android.gms.tagmanager.zzct.run(unavailable:-1)
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:423)
  at java.util.concurrent.FutureTask.run(FutureTask.java:237)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:154)
  at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:269)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:833)
```

第一行再次告诉我们线程被阻塞，那么我们就来看看造成阻塞的原因：
```
at com.google.android.gms.tagmanager.zzo.zza(unavailable:-1)
- waiting to lock <0x023ef607> (a com.google.android.gms.tagmanager.zzo) held by thread 1
```

我们看到这个线程正在等待由线程1（主线程）持有的另一个互斥体（称为 com.google.android.gms.tagmanager.zzo）。因此，我们看到线程1正在等待线程15，而线程15正在等待线程1。我们发现了死锁。

### 解决死锁
> Looking at the stack trace of thread 15 reveals that it does not pass through our code at all. By the look of the trace (note names such as ThreadPoolExecutor, FutureTask.run() and onSuccess) we could put forward an educated guess that this is part of an internal callback that is run when a backend call returns successfully, but sadly we cannot know for sure since the library is not open-source (Boo, Google!).

查看线程15的堆栈跟踪，发现它根本没有通过我们的代码。通过跟踪 trace（注意名称，例如 ThreadPoolExecutor，FutureTask.run() 和 onSuccess），我们可以提出一个有根据的猜测，这是内部回调的一部分，当后端调用成功返回时运行，但遗憾的是我们无法确切地知道，因为库不是开源的（Boo，Google！）。

看看主线程堆栈，尽管我们看到我们确实通过了自己的代码

让我们快速浏览一下有问题的代码：
```java
private void loadGTMContainer() {
    mAnalyticsService.loadGTMContainer(new GenericCallBack<ContainerHolder>() {
        @Override
        public void onSuccess(ContainerHolder result) {
            if (result != null) {
                LOGGER.debug("Google Tag Manager container loaded successfully");
                //manually refresh the container, just in case
                result.refresh();
                //...
            }
        }

        @Override
        public void onError(Exception e) {
            //...
        }
    });
}
```

堆栈似乎表明死锁发生在 result.refresh() 内部，事实上，上面的行（和上面的注释）似乎是可疑的。看起来刷新行是可选的（并导致 GTM 内部的问题），所以很有可能删除它也会消除死锁。但是如果不能可靠地重现错误，则无法确定。

现在是时候对“Google 跟踪代码管理器”，“死锁”和“刷新”这个主题进行一些研究。迟早你会从谷歌自己的论坛中找到这个[链接](https://productforums.google.com/forum/#!topic/tag-manager/wlPpNKPXvu8)。在那个发布于2014年的
晦涩难懂的论坛帖子中，你可以找到 GTM 库的一位开发人员，他承认在下列情况时会存在错误:

>  应用程序中的代码可能正在调用容器上的刷新，同时它正在保存从网络中检索到的容器的最新版本。[...] 减少死锁几率的一种方法是避免在初始加载期间调用刷新，以及在自动刷新发生时的时间间隔。入门文档没有详细说明，但容器会按照每隔12小时的间隔自动刷新。[...] 我们将在未来的版本中解决这个死锁。
