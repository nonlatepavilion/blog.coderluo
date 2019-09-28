---
title: Java线程堆栈分析
date: 2017-12-26 22:10:28
tags:
  - 线程堆栈
categories: JAVA
author: coderluo
---


> 不知觉间工作已有一年了，闲下来的时候总会思考下，作为一名Java程序员，不能一直停留在开发业务使用框架上面。老话说得好，机会是留给有准备的人的，因此，开始计划看一些Java底层一点的东西，尝试开始在学习的过程中写博客，希望和大家一起交流学习。

写在前面： 线程堆栈应该是多线程类应用程序非功能问题定位的最有效手段，可以说是杀手锏。线程堆栈最擅长与分析如下类型问题：

- 系统无缘无故CPU过高。
- 系统挂起，无响应。
- 系统运行越来越慢。
- 性能瓶颈（如无法充分利用CPU等）
- 线程死锁、死循环，饿死等。
- 由于线程数量太多导致系统失败（如无法创建线程等）。

## 如何解读线程堆栈

如下面一段Java源代码程序：

```java
package org.ccgogoing.study.stacktrace;
/** 
 * @Author: LuoChong400
 * @Description: 测试线程
 * @Date: Create in 07:27 PM 2017/12/08
 */
public class MyTest {

        Object obj1 = new Object();
        Object obj2 = new Object();

        public void fun1() {
            synchronized (obj1) {
                fun2();
            }
        }
        public void fun2() {
            synchronized (obj2) {
                while (true) { //为了打印堆栈，该函数堆栈分析不退出
                    System.out.print("");
                }
            }
        }
        public static void main(String[] args) {
            MyTest aa = new MyTest();
            aa.fun1();
        }
    }
```

在Idea 中运行该程序，然后按下CTRL+BREAK键，打印出线程堆栈信息如下：

```java
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.79-b02 mixed mode):

"Service Thread" daemon prio=6 tid=0x000000000c53b000 nid=0xca58 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" daemon prio=10 tid=0x000000000c516000 nid=0xd390 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" daemon prio=10 tid=0x000000000c515000 nid=0xcbac waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" daemon prio=6 tid=0x000000000c514000 nid=0xd148 runnable [0x000000000caee000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.read(SocketInputStream.java:152)
	at java.net.SocketInputStream.read(SocketInputStream.java:122)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:283)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:325)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:177)
	- locked <0x00000000d7858b50> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:154)
	at java.io.BufferedReader.readLine(BufferedReader.java:317)
	- locked <0x00000000d7858b50> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:382)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

"Attach Listener" daemon prio=10 tid=0x000000000ad4a000 nid=0xd24c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" daemon prio=10 tid=0x000000000c1a8800 nid=0xd200 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" daemon prio=8 tid=0x000000000ace6000 nid=0xcd74 in Object.wait() [0x000000000c13f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000000d7284858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	- locked <0x00000000d7284858> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" daemon prio=10 tid=0x000000000ace4800 nid=0xce34 in Object.wait() [0x000000000bf4f000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x00000000d7284470> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:503)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
	- locked <0x00000000d7284470> (a java.lang.ref.Reference$Lock)

"main" prio=6 tid=0x000000000238e800 nid=0xc940 runnable [0x00000000027af000]
   java.lang.Thread.State: RUNNABLE
	at org.ccgogoing.study.stacktrace.MyTest.fun2(MyTest.java:22)
	- locked <0x00000000d77d50c8> (a java.lang.Object)
	at org.ccgogoing.study.stacktrace.MyTest.fun1(MyTest.java:15)
	- locked <0x00000000d77d50b8> (a java.lang.Object)
	at org.ccgogoing.study.stacktrace.MyTest.main(MyTest.java:29)

"VM Thread" prio=10 tid=0x000000000ace1000 nid=0xd0a8 runnable 

"GC task thread#0 (ParallelGC)" prio=6 tid=0x00000000023a4000 nid=0xd398 runnable 

"GC task thread#1 (ParallelGC)" prio=6 tid=0x00000000023a5800 nid=0xcc20 runnable 

"GC task thread#2 (ParallelGC)" prio=6 tid=0x00000000023a7000 nid=0xb914 runnable 

"GC task thread#3 (ParallelGC)" prio=6 tid=0x00000000023a9000 nid=0xd088 runnable 

"VM Periodic Task Thread" prio=10 tid=0x000000000c53f000 nid=0xc1b4 waiting on condition 

JNI global references: 138

Heap
 PSYoungGen      total 36864K, used 6376K [0x00000000d7280000, 0x00000000d9b80000, 0x0000000100000000)
  eden space 31744K, 20% used [0x00000000d7280000,0x00000000d78ba0d0,0x00000000d9180000)
  from space 5120K, 0% used [0x00000000d9680000,0x00000000d9680000,0x00000000d9b80000)
  to   space 5120K, 0% used [0x00000000d9180000,0x00000000d9180000,0x00000000d9680000)
 ParOldGen       total 83456K, used 0K [0x0000000085800000, 0x000000008a980000, 0x00000000d7280000)
  object space 83456K, 0% used [0x0000000085800000,0x0000000085800000,0x000000008a980000)
 PSPermGen       total 21504K, used 3300K [0x0000000080600000, 0x0000000081b00000, 0x0000000085800000)
  object space 21504K, 15% used [0x0000000080600000,0x0000000080939290,0x0000000081b00000)
```

在上面这段堆栈输出中，可以看到有很多后台线程和main线程，其中只有main线程属于Java用户线程，其他几个都是虚拟机自动创建的，我们分析的过程中，只关心用户线程即可。

从上面的main线程中可以很直观的看到当前线程的调用上下文，其中一个线程的某一层调用含义如下：

```
at MyTest.fun1(MyTest.java:15)
    |     |     |              |
    |     |     |              +-----当前正在调用的函数所在的源代码文件的行号
    |     |     +------------当前正在调用的函数所在的源代码文件
    |     +---------------------当前正在调用的方法名
    +---------------------------当前正在调用的类名
```

另外，堆栈中有：`- locked <0x00000000d77d50b8> (a java.lang.Object)`语句，表示该线程已经占有柯锁<0x00000000d77d50b8>,尖括号中表示锁ID，这个事系统自动产生的，我们只需要知道每次打印的堆栈，同一个ID表示是同一个锁即可。每一个线程堆栈的第一行含义如下：

```
"main" prio=1 tid=0x000000000238e800 nid=0xc940 runnable [0x00000000027af000]
    |       |   |                       |           |           |
    |       |   |                       |           |           +--线程占用内存地址
    |       |   |                       |           +-----------线程的状态
    |       |   |                       +----线程对应的本地线程id号
    |       |   +-------------------线程id
    |       +--------------------------线程优先级
    +-------------------------------线程名称
    
其中需要说明的是，线程对应的本地线程id号，是指Java线程所对应的虚拟机中的本地线程。由于Java是解析型语言，执行的实体是Java虚拟机，因此Java语言中的线程是依附于虚拟机中的本地线程来运行的，实际上是本地线程在执行Java线程代码。
```
### 锁的解读

从上面的线程堆栈看，线程堆栈中包含的直接信息为：线程的个数，每个线程调用的方法堆栈，当前锁的状态。线程的个数可以直接数出来；线程调用的方法堆栈，从下向上看，即表示当前的线程调用了哪个类上的哪个方法。而锁得状态看起来稍微有一点技巧。与锁相关的信息如下：

- 当一个线程占有一个锁的时候，线程的堆栈中会打印--locked<0x00000000d77d50c8>
- 当一个线程正在等待其它线程释放该锁，线程堆栈中会打印--waiting to lock<0x00000000d77d50c8>
- 当一个线程占有一个锁，但又执行到该锁的wait()方法上，线程堆栈中首先打印locked，然后又会打印--waiting on <0x00000000d77d50c8>

### 线程状态的解读

借助线程堆栈，可以分析很多类型的问题，CPU的消耗分析即是线程堆栈分析的一个重要内容；

处于TIMED_WAITING、WAITING状态的线程一定不消耗CPU。处于RUNNABLE的线程，要结合当前代码的性质判断，是否消耗CPU。

- 如果是纯Java运算代码，则消耗CPU。
- 如果是网络IO，很少消耗CPU。
- 如果是本地代码，要结合本地代码的性质判断（可以通过pstack、gstack获取本地线程堆栈），如果是纯运算代码，则消耗CPU，如果被挂起，则不消耗CPU，如果是IO，则不怎么消耗CPU。


### 如何借助线程堆栈分析问题

**线程堆栈在定位如下类型的问题上非常有帮助：**

* 线程死锁的分析
* Java代码导致的CPU过高分析
* 死循环分析
* 资源不足分析
* 性能瓶颈分析

#### 线程死锁分析

死锁的概念就不做过多解释了，不明白的可以去网上查查；

两个或超过两个线程因为环路的锁依赖关系而形成的锁环，就形成了真正的死锁，如下为死锁喉打印的堆栈：

```java
Found one Java-level deadlock:
=============================
"org.ccgogoing.study.stacktrace.deadlock.TestThread2":
  waiting to lock monitor 0x000000000a9ad118 (object 0x00000000d77363d0, a java.lang.Object),
  which is held by "org.ccgogoing.study.stacktrace.deadlock.TestThread1"
"org.ccgogoing.study.stacktrace.deadlock.TestThread1":
  waiting to lock monitor 0x000000000a9abc78 (object 0x00000000d77363e0, a java.lang.Object),
  which is held by "org.ccgogoing.study.stacktrace.deadlock.TestThread2"

Java stack information for the threads listed above:
===================================================
"org.ccgogoing.study.stacktrace.deadlock.TestThread2":
	at org.ccgogoing.study.stacktrace.deadlock.TestThread2.fun(TestThread2.java:35)
	- waiting to lock <0x00000000d77363d0> (a java.lang.Object)
	- locked <0x00000000d77363e0> (a java.lang.Object)
	at org.ccgogoing.study.stacktrace.deadlock.TestThread2.run(TestThread2.java:22)
"org.ccgogoing.study.stacktrace.deadlock.TestThread1":
	at org.ccgogoing.study.stacktrace.deadlock.TestThread1.fun(TestThread1.java:33)
	- waiting to lock <0x00000000d77363e0> (a java.lang.Object)
	- locked <0x00000000d77363d0> (a java.lang.Object)
	at org.ccgogoing.study.stacktrace.deadlock.TestThread1.run(TestThread1.java:20)

Found 1 deadlock.
```

从打印的堆栈中我们能看到"Found one Java-level deadlock:",即如果存在死锁情况,堆栈中会直接给出死锁的分析结果.

当一组Java线程发生死锁的时候,那么意味着Game Over,这些线程永远得被挂在那里了,永远不可能继续运行下去。当发生死锁的线程在执行系统的关键功能时，那么这个死锁可能会导致整个系统瘫痪，要想恢复系统，临时也是唯一的规避方法是*将系统重启。然后赶快去修改导致这个死锁的Bug。*

注意：死锁的两个或多个线程是不消耗CPU的，有的人认为CPU100%的使用率是线程死锁导致的，这个说法是完全错误的。死循环，并且在循环中代码都是CPU密集型，才有可能导致CPU的100%使用率，像socket或者数据库等IO操作是不怎么消耗CPU的。















