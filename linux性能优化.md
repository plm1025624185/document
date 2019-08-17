linux性能优化
======

## CPU性能篇

### 系统平均负载

系统平均负载指的是平均的活跃进程数，那么最理想的就是每个CPU上都刚好运行着一个进程，这样每个CPU都得到了充分利用。比如当平均负载为2时，意味着什么呢？

* 在只有2个CPU的系统上，意味着所有的CPU都刚好被完全占用。
* 在4个CPU的系统上，意味着CPU有50%的空闲。
* 而在只有1个CPU的系统中，则意味着有一半的进程竞争不到CPU。

#### uptime命令中三个时间段平均负载的分析

平均负载最理想的情况是等于CPU个数。所以在评判平均负载时，**首先需要知道系统有几个CPU**，这可以从文件/proc/cupinfo中读取，例如：

```linux
grep 'model name' /proc/cpuinfo | wc -l
```

有了CPU个数，我们就可以判断出，当平均负载比CPU个数还大的时候，系统已经出现了过载。uptime命令提供了三个时间段的平均负载，可以方便我们分析**系统负载趋势**。

* 如果1分钟、5分钟、15分钟的三个值基本相同，或者相差不大，那就说明系统负载很平稳。
* 如果1分钟的值远小于15分钟的值，就说明系统最近1分钟的负载在减少，而过去15分钟内却有很大的负载。
* 如果1分钟的值远小于15分钟的值，就说明最近1分钟的负载在增加，这种增加有可能是临时性的，也有可能还会持续增加下去，所以就需要持续观察。一旦1分钟的平均负载接近或超过了CPU的个数，就意味着系统正在发生过载的问题，这时就得分析调查是哪里导致的问题，并要想办法优化。

举个例子，假设我们在一个单CPU系统上看到平均负载为1.73，0.60，7.98，那么说明在过去1分钟内，系统有73%的超载，而在15分钟内，有698%的超载，从整体趋势上看，系统的负载在降低。

#### 平均负载与CPU使用率

回到平均负载的定义上来，平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。所以，它不仅包含了**正在使用CPU**的进程，还包括**等待CPU**和**等待I/O**的进程。

而CPU使用率，是单位时间内CPU繁忙情况的统计，跟平均负载并不一定完全对应。比如：
* CPU密集型进程，使用大量CPU会导致平均负载升高，此时两者是一致的；
* I/O密集型进程，等待I/O也会导致平均负载升高，但CPU使用率不一定很高；
* 大量等待CPU的进程调度也会导致平均负载升高，此时的CPU使用率也会比较高。

#### 实战

**场景一：CPU密集型进程**

首先，我们在第一个终端运行stress命令，模拟一个CPU使用率100%的场景：

```Linux
stress --cpu 1 --timeout 600
```



# 性能相关命令

## uptime

**说明**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uptime命令一行中展示了后面的几个信息。当前的时间，系统运行了多长的时间，当前登录的用户数，过去1分钟的系统平均负载(system load averages)，过去5分钟的系统平均负载(system load averages)，过去15分钟的系统平均负载(system load averages)

**可选参数**

-p,&nbsp;&nbsp;&nbsp;&nbsp;--pretty&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以一个简洁的格式显示系统运行了多长时间

-h,&nbsp;&nbsp;&nbsp;&nbsp;--help&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显示帮助文本

-s,&nbsp;&nbsp;&nbsp;&nbsp;--since&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以yyyy-mm-dd HH:MM:SS的格式显示系统的启动时间

-V,&nbsp;&nbsp;&nbsp;&nbsp;--version&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显示版本信息并退出

**相关文件**

/var/run/utmp&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当前登录人的信息

/proc&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进程信息

# 专有名词解释

## 系统平均负载——System load average

**英文解释**

System load averages is the average number of processes that are either in a runnable or uninterruptable state.  A process in a  runnable  state  is  either using  the  CPU  or waiting to use the CPU.  A process in uninterruptable state is waiting for some I/O access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number of CPUs in a system, so a load average of 1 means a  single  CPU  system  is loaded all the time while on a 4 CPU system it means it was idle 75% of the time.

**中文解释**

简单来说，平均负载是指单位时间内，系统处于`可运行状态`和`不可中断状态`的平均进程数，也就是`平均活跃进程数`，它和CPU使用率并没有直接关系。

## 可运行状态的进程

**解释**

指正在使用CPU或者正在等待CPU的进程，也就是我们常用ps命令看到的，处于R状态（Running Man或Runnable）的进程。

## 不可中断状态的进程

**解释**

不可中断状态的进程则是正处于内核态关键流程中的进程，并且这些流程是不可中断的，比如最常见的是等待硬件设备的I/O响应，也就是我们在ps命令中看到的D状态（Uninterruptible Sleep，也称为Disk Sleep）的进程。

比如，当一个进程向磁盘读写数据时，为了保证数据一致性，在得到磁盘回复前，它是不能被其他进程或者中断打断的，这个时候的进程就处于不可中断状态。如果此时的进程被打断了，就容易出现磁盘数据与进程数据不一致的问题。所以，`不可中断状态实际上是系统对进程和硬件设备的一种保护机制`。