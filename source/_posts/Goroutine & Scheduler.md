---
title: golang之goroutine
date: 2020-08-10 20:40:22
index_img:
- /images/golang/goroutine.jpg
tags: 
- golang
categories:
- golang
---

### Goroutine & Scheduler

在上一篇的[并发模型和线程模型概述]()中我们了解到线程模型的分类。同时也知道了线程是 CPU 调度的实体，线程是真正在 CPU 上执行指令的实体。

每个程序启动的时候，都会创建一个初始进程，并启动一个线程。而线程可以取创建更多的线程。线程可以独立的执行。CPU 在这一层进行调度，而非进程。

在了解golang的scheduler之前，我们先了解下os scheduler。

#### os scheduler

调度是一个非常广泛的概念，在很多领域中都会使用调度这个术语，在计算机科学中，调度就是将一种任务分配给资源的方法。调度的核心就是对有限资源进行分配以实现最大化资源利用率或者降低系统的尾延迟，调度系统面对的就是资源的需求和供给不平衡的问题。任务可能是计算机任务，如线程、进程或者数据流，这些任务会被调度到硬件资源上执行。例如CPU等硬件设备

os scheduler 保证如果有可以执行的线程时，就不会让CPU闲着。并且还要保证，所有可执行的线程都看起来在同时执行。另外，OS scheduler 在保证高优先级的线程执行机会大于低优先级线程的同时，不能让低优先级的线程始终得不到执行的机会。OS scheduler 还需要做到迅速决策，以降低延时。

当然操作系统的调度器是十分复杂，它必须要考虑到底层的硬件结构，包括但不限于处理器数和内核数，cpu cache和NUMA。如果没有这些东西，调度器就没办法尽可能有效的工作。

#### Go scheduler

今天的Go语言调度器有着非常优异的性能，但是在最初的调度器不仅实现非常简陋，也无法支撑高并发的服务。调度器经过几个大版本的迭代才有今天的优异性能。对于之前的调度器不做过多的赘述，我们主要学习下在go1.12之后基于抢占式的调度器。其他的调度可以从[Go调度器](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/) 了解

##### G-P-M 模型

每个OS线程都有一个固定大小的内存块（一般是2M）来做栈，这个栈会用来存储当前正在被调用或者挂起的函数的内部变量。这个固定大小的栈同时很大又很小。因为2M的栈对于小小的Goroutine来说是很大的内存浪费，对于一些复杂的任务来说又显得太小。因此，Go语言做了自己的线程。Goroutine的栈初始仅为2K，随着任务执行按需增长，最大可达1G，完全由Go自己的调度器GoScheduler来调度。此外G C还会周期性的将不在使用的内存回收，收缩栈空间。

先来了解下G-P- M

- G：表示Gorutine，每个Goroutine对应一个G结构体，G存储Goroutine的运行堆栈、状态以及任务函数，可重用。
- M：Machine，OS线程抽象，代表着真正执行计算的资源，在绑定有效的P后，进入schedule循环。
- P：Processor，表示逻辑处理器。

在Go程序启动，主机上定义的每一个虚拟内核都会为它分配一个逻辑处理器(P)，比如我的机器上有4个物理核，每个物理核上有两个硬件线程（超线程），那么Go程序会分配4*2=8个逻辑处理器（P）。每个P会分配一个OS线程（M），这个现场时OS来处理，并且OS还负责把现场放置到Core上去执行。这意味着Go程序在我的机器上有8个可用的线程执行，每个线程单独连到一个P上。G并非执行体，每个G都要绑定到P才能被调度执行。

##### G-P-M模型调度

根据G- P- M模型我们知道，G的数量是可以成千上万个的。而P和M是和硬件相关的。所以延伸出两种用来保存G的任务队列：一种是Global任务队列，一种是每个P维护Local任务队列。

当通过go 关键字创建一个新的Goroutine的时候，它会优先被放入P的本地队列。为了允许Goroutine，M需要持有一个P，接着M会启动一个OS线程，循环从P的本地队列取出一个Goroutine并执行。当M执行完了当前P的Local队列里所有的G后，P也不会就这个闲下来，它会先尝试从Global队列寻找G来执行，如果Global队列为空，它会随机挑选另外一个P，从它的队列里拿走一半的G到自己的队列中执行。

如果一切正常，调度器会以上述的那种方式顺畅的运行，但是这个世界没这么美好，总有意外发生，以下分析Goroutine在两种例外情况下的行为

Go runtime 会在下面的 Goroutine 被阻塞的情况下运行另外一个 Goroutine：

- blocking syscall (for example opening a file)
- network input
- channel operations
- primitives in the sync package

这四种场景又可归类为两种类型：

1. 用户态阻塞/唤醒

   当 Goroutine 因为 channel 操作或者 network I/O 而阻塞时（实际上 Golang 已经用 netpoller 实现了 Goroutine 网络 I/O 阻塞不会导致 M 被阻塞，仅阻塞 G，这里仅仅是举个栗子），对应的 G 会被放置到某个 wait 队列(如 channel 的 waitq)，该 G 的状态由`_Gruning`变为`_Gwaitting`，而 M 会跳过该 G 尝试获取并执行下一个 G，如果此时没有 runnable 的 G 供 M 运行，那么 M 将解绑 P，并进入 sleep 状态；当阻塞的 G 被另一端的 G2 唤醒时（比如 channel 的可读/写通知），G 被标记为 runnable，尝试加入 G2 所在 P 的 runnext，然后再是 P 的 Local 队列和 Global 队列。

2. 系统调用阻塞

   当 G 被阻塞在某个系统调用上时，此时 G 会阻塞在`_Gsyscall`状态，M 也处于 block on syscall 状态，此时的 M 可被抢占调度：执行该 G 的 M 会与 P 解绑，而 P 则尝试与其它 idle 的 M 绑定，继续执行其它 G。如果没有其它 idle 的 M，但 P 的 Local 队列中仍然有 G 需要执行，则创建一个新的 M；当系统调用完成后，G 会重新尝试获取一个 idle 的 P 进入它的 Local 队列恢复执行，如果没有 idle 的 P，G 会被标记为 runnable 加入到 Global 队列。

更多的调度过程可参考[理解golang调度之二 ：Go调度器](https://juejin.cn/post/6844903846825705485)含图解

#### 工作负载

如何知道无序执行(并发)是可行的呢？了解所处理问题的工作负载(workload)是一个起点。有两种类型的工作负载在并发的时候要考虑到。

- **CPU密集(CPU-Bound)**：这种工作负载情况不会有Goroutines自动切换到waiting状态的情况，也不会有自动从waiting状态切到其他状态的情况。这种情况发生在进行持续计算的时候。线程计算Pi值就是CPU-Bound。
- **IO密集(IO-Bound)**：这种工作负载会导致Goroutines自动进入等待状态。这种工作发生在持续地请求网络资源、或者是进行系统调用、或者是等待事件发生的情况。一个Goroutines需要读文件就是IO-Bound。我把同步事件(mutexes，atomic)类似导致Goroutine等待的情况归到此类。

**cpu-bound的工作负载，你需要并行去使用并发**。一个单独的OS/硬件线程处理多个Goroutines效率很低，因为Goroutines在这个工作负载里不会主动进入或者是离开等待状态。Goroutines数多于OS/硬件线程数的时候会降低工作负载的执行速度，因为从OS线程换上或者是换下Goroutines会有延迟(切换的时间)。上下文切换会在workload里创建出“一切都停止”事件，因为在切换的时候你的所有workload都不会执行。

**在IO-Bound的workloads里，你不需要并行去使用并发**。一个单独OS/硬件线程可以有效率地处理多个Goroutines，因为Goroutines作为它自己workload的一部分可以自动进入或者离开等待状态(waiting)。Goroutines数量多于OS/硬件线程数可以加速workload的执行,因为Goroutines在OS线程上切换不会创建“一切都停止”事件。你的workload会自然停止并且这会让一个不同的Goroutine去有效率地使用相同的OS/硬件线程，而不是让OS/硬件线程空闲下来。

#### 并发和并行

| 类型      | 线程数 | 结论                                       | 原因                                                         |
| --------- | ------ | ------------------------------------------ | ------------------------------------------------------------ |
| CPU密集型 | 1      | 起一个goroutine，性能要好于起多个goroutine | 1个OS线程中，无法并行计算。多个goroutine 上下文切换会增加性能损耗 |
| CPU密集型 | 大于1  | 起多个goroutine，性能要好于起一个goroutine | 多个OS线程中可以并行计算。收益大于goroutine上下文切换        |
| IO密集型  | 1      | 起多个goroutine，性能高于起一个goroutine   | 1个OS线程中，多个goroutine都能有效共享一个OS线程，每个goroutine能够自动进行上下文切换，OS线程一直有事情做 |
| IO密集型  | 大于1  | 起多个goroutine，性能高于起一个goroutine   | 多个OS线程和1个OS线程相比差距不大，额外的OS/硬件线程并没有提供更好的性能。这是因为更多的goroutine会进入等待状态。 |

该表格结论来自于[理解golang调度之三：并发](https://juejin.im/post/6844903847568080904)

参考：

https://qcrao.com/2019/09/02/dive-into-go-scheduler/

https://www.infoq.cn/article/XF6v3Vapqsqt17FuTVst

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/

https://www.cnblogs.com/jiujuan/p/12735559.html

