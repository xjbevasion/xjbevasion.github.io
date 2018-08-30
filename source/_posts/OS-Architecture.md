---
title: OS-Architecture
date: 2018-08-28 01:21:32
categories: "MacOS Internals"
tags: "Architecture"
---
系统的架构是比较复杂的，在苹果的开发文档中，展示了一种[针对开发人员的分层方式](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/OSX_Technology_Overview/About/About.html)：

# Layers of OS
|  | 
| --- |
| Cocoa(Application) |
| Media |
| Core Services |
| Core OS |
| Kernel and Device Drivers |


* Cocoa（Application）
 Cocoa框架层或者应用程序层，用户UI，包括了用于开发界面程序的框架集合
* Media
媒体层，包含了图像、声音等开发需要的框架，苹果列出了有AVFoundation,CoreAudio, CoreImage 和 CoreAnimation.包括一些下层的支持-OpenAL,OpenGL,Quartz.
* Core Services
核心服务层，提供了系统安全、底层、内部数据访问及存储的支持接口，不直接影响UI。包括了 Foundation framework(NS* APIs) 和 CoreFoundation(CF* APIs),还有一些其他的框架。
* Core OS
核心系统层。提供了系统基础的OS APIs,包含加速器、异常处理、网络接口、系统配置等框架
* Kernel and Device Drivers
内核与驱动层。包括XNU和所有类型的内核扩展，提供了开发设备驱动和内核扩展所需的一些框架。


其中Core OS 与Kernel and Devices Drivers 一起也被称为Darwin层。Darwin 是OSX的核心操作系统，是直接管理硬件的部分，包括系统内核与shell环境（System utilities）。

# Darwin
Darwin的XNU内核中包扩了一个BSD层和Mach层
## Mach（I/Okit, Driver）
以mach为独立的微内核，处于底层，为上层提供最简单的原义，同时为了保持微内核的优势，及稳定性，提供的特性比较简单、基础。
* 内存保护。一个进程如果意外的将数据写入系统的其它进程，会导致数据丢失或系统崩溃。Mach避免这种情况的发生，Mach让系统的一部分不会破坏其它的部分。
* 抢占式多任务。Mach监视CPU，调度任务（task）的运行，保证系统的高效率和资源利用。由系统决定一个任务的执行时间，而不是另一进程 
* 虚拟内存。每个进程有自己的唯一的地址空间。32位应用程序有4GB的地址空间，64位应用程序有18EB的地址空间。Mach处理从任务中虚内存到物理内存的映射。在任意时刻，只有一部分虚内存实际驻留在物理内存中。物理内存在需要时被换入（page in）。Mach使用内存对象（memory objects）的概念让一个任务映射一部分内存，逆映射，并送入另一任务 
* 实时支持。对时间敏感的应用程序保证低延迟。

Mach也支持合作多任务（cooperative multitasking），强占式线程（preemptive threading）和合作线程（cooperative threading）。
## BSD(Berkeley Software Distribution) (filesystem, NKE)
为了通过unix验证，及提供丰富的系统编程接口，在mach层之上，分别提供了两个子系统， BSD, POSIX（通用系统）系统.主要包括fileSystem 和 Network Environment。
* 进程模型（进程ID，信号）
* 基础的安全策略（文件权限设定和用户、组ID设定）
* 线程支持（POSIX线程）
* 网络支持（BSD套接字）
