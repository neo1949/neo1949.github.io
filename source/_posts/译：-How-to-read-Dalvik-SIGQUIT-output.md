---
title: 【译】 How to read Dalvik SIGQUIT output
date: 2018-04-09 21:01:04
tags:
categories:
---

原文：[How to read Dalvik SIGQUIT output](http://elliotth.blogspot.com/2012/08/how-to-read-dalvik-sigquit-output.html)

限于本人英语水平，且对原文中的相关专业术语缺乏了解，因此翻译难免会有错误和歧义，欢迎大家指教。在翻译时对译文做了分段处理，并在译文相关地方添加了一些辅助说明，不需要译文的请直接点击上面的链接（需科学上网）查看原文。

<!-- more -->

翻译文章的目的是通过翻译的过程学习相关知识，而且可以做个备份，日后再需要使用的时候可以很方便的查阅。英文水平差无关紧要，有谷歌翻译可以帮助我们解决大部分问题，再加上自己的一点理解即可。闲话不再多说，下面开始进入正文。

## 引言
> If you're a long-time Java developer you're probably used to sending SIGQUIT to a Java process (either via kill -3 or hitting ctrl-\) to see what all the threads are doing. You can do the same with Dalvik (via adb shell kill -3), and if you're ANRing the system server will be sending you SIGQUIT too, in which case the output will end up in /data/anr/traces.txt (see the logcat output for details).

如果你是一位长期从事 Java 开发的人员，那么你可能已经习惯于将 SIGQUIT 发送给 Java 进程（通过 <code>kill -3</code> 或者 敲击 <code>Ctrl-\\</code>）来查看所有线程正在执行的操作。你可以使用 Dalvik （通过 <code>adb shell kill -3</code>）做同样的事情，系统服务也会向你发送 SIGQUIT，这种情况下输出将会被保存到 <code>/data/anr/traces.txt</code> 中（请参阅 logcat 输出获取详细信息）。


> Anyway, I've found that very few people actually know what all the output means. I only knew a few of the more important bits until I became a Dalvik maintainer. This post should hopefully clear things up a little.

总之，我发现很少有人真正知道所有的输出是是什么意思。在成为 Dalvik 维护人员之前，我也只是知道一些更重要的部分。希望这篇文章能够澄清一些事情。

## 示例及说明
> To start with, here's an example from my JNI Local Reference Changes in ICS post:

首先，这是我在 ICS post 中 JNI Local Reference Changes 的一个例子：
```java
"Thread-10" prio=5 tid=8 NATIVE
  | group="main" sCount=0 dsCount=0 obj=0xf5f77d60 self=0x9f8f248
  | sysTid=22299 nice=0 sched=0/0 cgrp=[n/a] handle=-256476304
  | schedstat=( 153358572 709218 48 ) utm=12 stm=4 core=8
  at MyClass.printString(Native Method)
  at MyClass$1.run(MyClass.java:15)
```

> Ignore the Java stack trace for now. If there's demand, I'll come back to that later, but there's nothing interesting in this particular example. Let's go through the other lines...

现在请暂时忽略 Java 堆栈跟踪。 如果有需求，我会在稍后说明，但这个例子中没有什么令人关注的。我们来看看其它行的信息：


> First, though, a quick note on terminology because there are a lot of different meanings of "thread" that you'll have to keep clear. If I say Thread, I mean java.lang.Thread. If I say pthread, I mean the C library's abstraction of a native thread. If I say native thread, I mean something created by the kernel in response to a clone(2) system call. If I say Thread*, I mean the C struct in the VM that holds all these things together. And if I say thread, I mean the abstract notion of a thread.

首先是关于术语的简要说明，因为你必须明白 **thread** 有许多不同的含义。 如果我说 <code>Thread</code>，我的意思是 <code>java.lang.Thread</code>。如果我说 <code>pthread</code>，我的意思是C库的本地线程的抽象。如果我说本地线程，我的意思是内核创建的响应克隆（2）系统调用的东西。如果我说 <code>Thread *</code>，我的意思是 VM 中将所有这些东西放在一起的 C 结构。如果我说 <code>thread</code>，我的意思是线程的抽象概念。

### 第一行参数说明
```
"Thread-10" prio=5 tid=8 NATIVE
```

> The thread name comes first, in quotes. If you gave a name to a Thread constructor, that's what you'll see here. Otherwise there's a static int in Thread that's a monotonically increasing thread id, used solely for giving each thread a unique name. These thread ids are never reused in a given VM (though theoretically you could cause the int to wrap).

首先是引号中的线程名称。如果你向 <code>Thread</code> 的构造函数传递了一个名称，那么你会在这里看到。否则，<code>Thread</code> 中有一个静态 int，它是一个单调递增的线程 ID，仅用于给每个线程一个唯一的名称。这些线程 ID 不会在给定的虚拟机中重用（尽管理论上你可能会导致 int 溢出）。


> The thread priority comes next. This is the Thread notion of priority, corresponding to the getPriority and setPriority calls, and the MIN_PRIORITY, NORM_PRIORITY, and MAX_PRIORITY constants.

接下来是线程优先级。这是 <code>Thread</code> 的优先级概念，对应于 <code>getPriority</code> 和 <code>setPriority</code> 调用，以及 <code>MIN_PRIORITY</code>，<code>NORM_PRIORITY</code> 和 <code>MAX_PRIORITY</code> 常量。

<code>java.lang.Thread</code> 中对于三个常量的定义如下：
```java
/**
 * The minimum priority that a thread can have.
 */
public static final int MIN_PRIORITY = 1;

/**
 * The default priority that is assigned to a thread.
 */
public static final int NORM_PRIORITY = 5;

/**
 * The maximum priority that a thread can have.
 */
public static final int MAX_PRIORITY = 10;
```


> The thread's thin lock id comes next, labelled "tid". If you're familiar with Linux, this might confuse you; it's not the tid in the sense of the gettid(2) system call. This is an integer used by the VM's locking implementation. These ids come from a much smaller pool, so they're reused as threads come and go, and will typically be small integers.

接下来是标记为 <code>tid</code> 的线程的 **thin lock** ID。如果你熟悉 Linux，这可能会让你困惑；它不是 gettid 系统调用意义上的 tid。这是 VM 锁定实现所使用的整数。这些 ID 来自一个小得多的池，所以它们在线程来来去去时被重用，并且通常是小整数。

关于什么是 thin lock id，请参考 [Understanding Threads and Locks](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/thread_basics.html#wp1090499) 中对于 Locks 的说明。


> The thread's state comes last. These states are similar to, but a superset of, the Thread thread states. They can also change from release to release. At the time of writing, Dalvik uses the following states (found in enum ThreadStatus in vm/Thread.h):

最后是线程的状态。这些状态与 <code>Thread</code> 的线程状态类似，但是是超线程状态。它们从一个版本到另一个版本可能发生变化。在撰写本文时，Dalvik使用以下状态（可在 vm/Thread.h 的枚举 ThreadStatus 中找到）：

```c
/* these match up with JDWP values */
THREAD_ZOMBIE       = 0,        /* TERMINATED */
THREAD_RUNNING      = 1,        /* RUNNABLE or running now */
THREAD_TIMED_WAIT   = 2,        /* TIMED_WAITING in Object.wait() */
THREAD_MONITOR      = 3,        /* BLOCKED on a monitor */
THREAD_WAIT         = 4,        /* WAITING in Object.wait() */
/* non-JDWP states */
THREAD_INITIALIZING = 5,        /* allocated, not yet running */
THREAD_STARTING     = 6,        /* started, not yet on thread list */
THREAD_NATIVE       = 7,        /* off in a JNI native method */
THREAD_VMWAIT       = 8,        /* waiting on a VM resource */
THREAD_SUSPENDED    = 9,        /* suspended, usually by GC or debugger */
```


> You won't see ZOMBIE much; a thread is only in that state while it's being dismantled. RUNNING is something of a misnomer; the usual term is "runnable", because whether or not the thread is actually scheduled on a core right now is out of the VM's hands. TIMED_WAIT corresponds to an Object.wait(long, int) call. Note that Thread.sleep and Object.wait(long) are currently both implemented in terms of this. WAIT, by contrast, corresponds to a wait without a timeout, via Object.wait(). MONITOR means that the thread is blocked trying to synchronize on a monitor, Either because of a synchronized block or an invoke of a synchronized method (or theoretically, on a call to JNIEnv::MonitorEnter).

你不会经常看到 **ZOMBIE** 状态，一个线程只有在被拆除时才会处于该状态。**RUNNING** 不是很贴切，通常术语是“running”，因为无论线程是否在一个核心上，现在已经不在 VM 的掌控中了。**TIMED_WAIT** 对应于 <code>Object.wait(long，int)</code> 调用。请注意， <code>Thread.sleep</code> 和 <code>Object.wait(long)</code> 目前都是以这种方式实现的。相反，**WAIT** 对应于通过 <code>Object.wait()</code> 实现的没有超时的等待。**MONITOR** 意味着线程被阻塞，试图在监视器上进行同步，或者是因为同步块或调用同步方法（或理论上在调用 <code>JNIEnv :: MonitorEnter</code> 时）。


> The INITIALIZING and STARTING states are aspects of the current (at the time of writing) implementation of the thread startup dance. As an app developer, you can probably just chunk these two as "too early to be running my code". NATIVE means that the thread is in a native method. VMWAIT means that the thread is blocked trying to acquire some resource that isn't visible to managed code, such as an internal lock (that is, a pthread_mutex). SUSPENDED means that the thread has been told to stop running and is waiting to be allowed to resume; as the comment says, typically as an app developer you'll see this because there's a GC in progress or a debugger is attached.

**INITIALIZING** 和 **STARTING** 状态是当前（编写时）线程启动舞蹈的实现方面。 作为一名应用程序开发人员，您可能只是将这两者分成“过早运行我的代码”。**NATIVE** 意味着线程处于本地方法。**VMWAIT** 意味着线程被阻塞，试图获取某些托管代码不可见的资源，例如内部锁（即pthread_mutex）。**SUSPENDED** 意味着线程已被告知停止运行，并且正在等待被允许恢复; 正如评论所说，通常作为应用程序开发人员，你会看到这一点，因为有 GC 正在进行或者附加了调试程序。


> Not shown in this example, a daemon thread will also say "daemon" at the end of the first line.

在这个例子中没有展示，一个守护线程通常会第一行的末尾使用 <code>daemon</code> 说明。


### 第二行参数说明
```
| group="main" sCount=0 dsCount=0 obj=0xf5f77d60 self=0x9f8f248
```

> The Thread's containing ThreadGroup name comes next, in quotes.

接下来是用引号括起来的包含 ThreadGroup 的线程名称。


> The sCount and dsCount integers relate to thread suspension. The suspension count is the number of outstanding requests for suspension for this thread; this is sCount. The number of those outstanding requests that came from the debugger is dsCount, recorded separately so that if a debugger detaches then sCount can be reset appropriately (since there may or may not have been outstanding non-debugger suspension requests, we can't just reset sCount to 0 if a debugger disconnects).

<code>sCount</code> 和 <code>dsCount</code> 整数与线程挂起有关。暂停计数是此线程未决请求的数量，这是 <code>sCount</code>。来自调试器的未完成请求的数量是 <code>dsCount</code>，分开记录，以便如果调试器分离，则可以适当地重置sCount（因为可能有或可能没有未完成的非调试器暂停请求，如果调试器断开，我们不能只把 <code>sCount</code> 重置为0）。


### 第三行参数说明
```
sysTid=22299 nice=0 sched=0/0 cgrp=[n/a] handle=-256476304
```

### 第四行参数说明
```
schedstat=( 153358572 709218 48 ) utm=12 stm=4 core=8
```

## 参考文章
[Understanding Threads and Locks - Oracle](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/thread_basics.html)
