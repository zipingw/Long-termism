# MIT 6.824

## Lecture 1 :Introduction

### What（什么是分布式系统）

多个计算机 通过网络 合作，keywords : multiple networked cooperating computers

### Why（为什么需要分布式系统）

连接在物理空间中分离的机器==>sharing

1. Increase capacity through parallelism
2. Tolerate fault(当某些机器出现错误)
3. Achieve security（隔离机器在不同的物理空间，使得不会被同时攻击）

### Historical context

- 伴随着局域网的出现（1980s） Athena 、AFS

  互联网规模的分布式系统主要是DNS（域名服务系统）和email

- Data centers & Big Website (1990s) 80年代政府同意互联网上的商业行为 促使其繁荣

  Web search，shopping ，大量的数据和大量的用户的出现使得无法使用单独的机器处理

- Cloud computing (2000s) 随着数据中心的出现，很多计算在云上完成而非本地，云提供商开始大量建设基础设置，为了使得客户将其分布式系统扩展至更大范围机器上，实现高并行，高性能

- Current State : Active!!! 在学术界（研究领域）和工业界（开发领域）都是如此

### Challenges

两件事情导致其困难：

1. many concurrent parts(并行性)
2. must deal with partial failure(处理某些机器故障时的情况)

这两件事情合并在一起让理解系统变得困难（比如脑裂）

3. tricky realize the performance benefit

### Why take 6.824

1. Interesting : hard problems but powerful solutions
2. Used in real world
3. Active area in research(是个活跃的研究领域，有很多悬而未决的问题)
4. Handsome (实验中搭建系统会学到另一种编程技巧类型) 

### Course structure

- Lectures : big ideas
- Papers : case study (回答一个问题并提出一个问题)
- Labs : 4 main labs (实验有递进性，都提供了test cases，许多进程模拟多个机器，实验有多不同的RPC库)
  - map reduce , 构建自己的map reduce library
  - Raft 重点在复制 存在故障和分区网络情况下 用Raft实现复制，构建Raft库实现复制服务
  - 复制键值服务 Replicated K/V service
  - Sharded K/V service 分区

### Focus on Infrastructure

基础设施和应用程序，不关心应用程序

- Storage 存储基础架构（K/V service , file system）
- Computation（编排和构建应用程序）
- Communication (6.829) ： RPC远程过程调用（最多一次恰好一次至少一次）

### Main Topics

- Fault Tolerance
  - availability (replication)
  - recoverability (logging/transaction)
- Consistency
-  Performance
  - throughput(吞吐量)
  - latency(时延)：tail latency
- Implementation
  - 如何管理并发
  - 如何实现RPC（远程过程调用）

 上述三个其实也就是 分布式系统中的不可能三角

### Case Study : Map Reduce

#### Context : Google     

- Computation takes multi-hours on TB level data
- goal : easy for non-experts
- Approach（使得编写者无需关注分布式系统中的问题，代价是并不通用） :
  - map function & reduce function
  - Map Reduce 框架处理分布式

#### Abstract view

- Map get some key-value pairs 

  <img src="C:\Users\zipin\AppData\Roaming\Typora\typora-user-images\image-20230920110510042.png" alt="image-20230920110510042" style="zoom:50%;" />

- Reduce

   <img src="C:\Users\zipin\AppData\Roaming\Typora\typora-user-images\image-20230920110731579.png" alt="image-20230920110731579" style="zoom:50%;" />

- 这个框架中的EmitIntermediate和Emit使得程序员只要写简单的顺序代码就能实现分布式应用

#### Implementation

- Coordinator
- Master
- Worker

tackle fault

- coordinator fail
- slow workers

实现时要使得使用map reduce函数的程序员完全不用考虑分布式的特性

## Lecture 2 : RPC and Threads（Go）

### Why Go?

1. Good support for threads and RPC，这对分布式系统很重要
2. garbage collection 多个线程共享一个结构体或变量，不用考虑最后一个引用该内存的线程
3. type safe 
4. compile，编译型语言

### 内容概述

- 介绍线程和RPC
- 讨论使用线程编程的不同方面 并发编程
- 爬虫，channel mutex

### Thread of execution

go run会创建一个执行线程，即主线程

go语言有一些原语操作实现对线程的控制

- start/go  
- exit
- stop (线程向channel中写入一些内容，但是channel中没有接收者，线程将被阻塞)
- resume

### Why threads?

用线程来表达并行，下面举两三种不同类型的并发

- I/O concurrency
- multicore  parallelism 多核并行性
- Convenience (比如休眠200ms)

### Threads challenges

- Race conditions （竞态挑战）
  - 两个线程同时进行 n = n + 1操作，从寄存器层面来看，可能指令的执行是交叉的，导致两个线程最终从寄存器中读取到的值都是错误的
  - 通常不会出现这样的错误，只在某些时刻发生，这也导致了错误的隐蔽性
  - 解决方法：
    - Acoiding sharing
    - Use locks
    - Race  detect `go --race`
- Coordination
  - Channels or Condition variables
- Deadlocks
  - ch <- true 只有一个线程时没有线程读取通道导致产生死锁

### Go and chanllenges

Two plans:

- channels(no sharing) 适合不共享内存的场景
- locks + condition variables 必须共享内存时使用该方案

讨论一下条件变量：



上述程序中存在Race condition，可以使用锁机制修改程序



GO中匿名函数中使用到的变量， 函数中使用的不是函数中声明的任何变量，将会指向外部作用域的变量，即静态作用域

互斥锁的作用域是基本块



这里可以学习到有趣的匿名函数定义与调用，匿名函数是指不事先声明而是将”定义“与”使用“同时进行，如上图中代码块所示，go func(){}()定义了匿名函数，可以在第一个小括号中定义参数，可以在第二个小括号中传入参数

如果不将 i 作为参数传入而是直接在函数中使用 i ，则会导致线程中使用 i 时得到的值是那一时刻的 i 值



这里是加入了 condition 来进一步优化，condition可以被视为两个线程之间的协调原语，其实就是信号量机制实现互斥与同步



这是用过通道解决该问题，但是使用通道会有一些问题，比如该程序中如果前五个线程都投了同意票，那后五个线程将会因为写入通道的内容无线程读取而被阻塞

### Crawler































