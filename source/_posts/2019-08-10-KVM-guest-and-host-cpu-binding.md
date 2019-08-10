---
title: KVM guest and host cpu binding
date: 2019-08-10 08:47:39
tags:
- KVM
- Virtualization
---

kvm虚拟出来的虚拟机（vm）是运行在单独的一个逻辑cpu还是可以分别在各个cpu之间运行？虚拟机cpu（vcpu）是什么概念？物理机（host）怎么看待kvm和vcpu? 为了搞懂这个概念我们还是要回到命令行中看。举例说明：
我这里有一个虚拟机叫core8，它含有8个虚拟cpu它的进程编号是20736。
<!-- more -->
就是说core8在host看来就是一个进程而已，这个集成的编号是20736.那么现在提出一个问题，这个core8的8个vcpu是怎么个情况呢？在哪里运行呢？这时还是得借助命令行。我们在host里使用ps指令，但是不能单纯用ps，还要借助于参数： ps -eL （e的意思是打印所有进程，L的意思是连县城也不放过）。我这里只显示一下和我们的20736进程相关的信息：

kvm虚拟机vcpu资源绑定_第1张图片

你会看到和20736相关的有九行，那么这九行是什么呢？首先第一列都是20736，第二列里只有第一行是20736，后面的都不是。那么我们这时就应该明白了，对于host来说，kvm虚拟机是一个进程（20736），虚拟机的vcpu都是这个进程衍生出来的线程。这就是为什么除了20736还有另外八行的原因。

那么我们接着询问，这八个线程是跑在同一个逻辑cpu里吗？为了回答这个问题，我们接着做实验：

还是借助于ps指令： `ps -eLo ruser,pid,ppid,lwp,psr| awk ‘{if($5==1) print $0}’`
解释为：ps命令显示当前系统的进程信息的状态，它的“-e”参数用于显示所有的进程，“-L”参 数用于将线程（LWP，light-weight process）也显示出来，“-o”参数表示以用户自定义的格式输出（其中“psr”这列表示当前分配给进程运行的处理器编号，“lwp”列表示线程的 ID，“ruser”表示运行进程的用户，“pid”表示进程的ID，“ppid”表示父进程的ID，）。结合ps和 awk工具的使用，是为了分别打印出来运行在不同的逻辑cpu上的进程线程情况。上面的指令就是打印出1号（从0开始编号）cpu的进行线程情况，我们这里只列出和我们相关的：

这时看到，20736号进程衍生出来的线程只有一部分运行在逻辑cpu1上，其它的线程在其它的cpu上了。

这时就大概明白了，不同的vcpu只是不同的线程，而不同的线程是跑在不同的cpu上的。

Qemu/kvm为客户机提供一套完整的硬件系统环境，在客户机看来其所拥有的cpu即是vcpu（virtual CPU）。在KVM环境中，每个客户机都是一个标准的Linux进程（qemu进程），而每一个vCPU在宿主机中是Qemu进程派生的一个普通线程。

KVM中的一个客户机作为一个用户空间进程（qemu-kvm）运行的，它和其他普通的用户进程一样由内核来调度使其运行在物理cpu上，不过它由KVM模块控制。多个客户机就是宿主机中的多个QEMU进程，而一个客户机的多个vCPU就是一个QEMU进程中的多个线程。在客户机系统中，同样分别运行着客户机的内核和客户机的用户空间应用程序。
疑问：

列表 KVM环境下的smp的架构是如何处理时限的，也就是cores * 2，socket * 2 ，threads * 2与 cores * 1，socket * 8，thread * 1 有什么本质的区别？

因为为虚拟机提供计算的vcpu在宿主机上实际为一个线程，线程上并不会对cores、sockets或者threads做明确区分，也就是整体设置222 和 181是没区别的。不过在虚拟机内部，可能会有系统倾向于（服务器的设别程度）多物理插槽，或者多物理核心。（之前遇到的win2k8（非Datacenter）的情况就是，无法识别超过8个的vcpu，但是设置成261 即可实现12个vcpu的服务器

### 进程的处理器亲和性和VCPU绑定

什么是进程亲和性：简单的一句话就是我这个Linux进程到底可以在哪几个cpu上做负载均衡。

KVM虚拟机是一个普通的linux进程，vcpu是一个线程，我们可以在宿主机上将vcpu线程对应的tid绑定到指定的cpu上。

#### 应用场景
用户希望把虚拟机的VCPU绑定在特定物理CPU上，VCPU只在绑定的物理CPU上调度，达到隔离VCPU并提升虚拟机性能的目的。如果没有作VCPU绑定，则虚拟机的VCPU可以在所有物理CPU上调度。

绑定vcpu到指定的cpu上，的确会提高性能，因为在vcpu的线程在物理cpu做负载均衡的时候，会有一些必要的数据结构初始化（vmlaunch）相对于VM-Entry来说是比较奢侈的，加上cache的命中，性能必然会有所提高，但破坏了负载均衡。当绑定在同一cpu上的两个vcpu同时高负载的时候，性能就会大打折扣，而其他的cpu也没有得到充分的利用。

在KVM环境中，一般并不推荐手动设置qemu进程的处理器亲和性来绑定vCPU，但是，在非常了解系统硬件架构的基础上，根据实际应用的需求，可以将其绑定到特定的CPU上，从而提高客户机中的CPU执行效率或实现CPU资源独享的隔离性。

为了实现这个功能，你首先得会`taskset`命令，直观上来说，taskset就是设置任务，也就是制定任务运行的情况，是一个很好用的工具。
```SHELL
　taskset -p [mask] pid
```
taskset绑定进程到某个CPU是很方便的：
#taskset -pc 0,1 1249
这会绑定1249进程到0号跟1号cpu上。

#cat /proc/1249/status
Cpus_allowed: 3
Cpus_allowed_list: 0-1

重新绑定下：

#taskset -pc 1 1249
#cat /proc/1249/status
Cpus_allowed: 2
Cpus_allowed_list: 1

注意这里的Cpu_allowed用的是 二进制掩码，3的二进制是11，2的二进制是10。前一个表示可在两个CPU上运行，第二个表示仅在第二个CPU上运行。
那么我们这里就可以使用taskset了，只需把这九个线程都绑定在同一个cpu上即可。假设我们把这个虚拟机绑定到1号cpu上：
taskset -p 2 20736
taskset -p 20740
taskset -p 20741
......
taskset -p 20747

ok,这时你再运行ps -eLo ruser,pid,ppid,lwp,psr| awk ‘{if($5==1) print $0}’，会看到形如下面的结果：

那么也就是完成了我们的虚拟机绑定任务，为了验证一下是否真正实现了绑定，我们在core8虚拟机里运行一个死循环，然后看host里的 top指令的结果：


我们可以清楚的看到1号cpu的利用率100%，而其它的cpu基本上没用到，这说明我们的绑定是成功的，完成了客户提出的需求。

Taskset命令设置某虚拟机在某个固定cpu上运行

设置某个进程pid在某个cpu上运行：

[root@test~]# taskset -p 100 95090
pid 95090's current affinity mask: 1
pid 95090's new affinity mask: 100

**解释**：设置95090这个进程，在cpu8上运行

95090是我提前用ps –aux|grep “虚拟机名” 找到的虚拟机进程id。

#### vcpupin命令设置虚拟机某个vcpu在某个固定cpu上运行

vcpupin的命令如下：
virsh vcpupin 4 0 8：绑定domain 4的vcpu 0 到物理CPU8
taskset和vcpupin区别
Taskset是以task（也就是虚拟机）为单位，也就是以虚拟机上的所有cpu为一个单位，与物理机上的cpu进行绑定，它不能指定虚拟机上的某个vcpu与物理机上某个物理cpu进行绑定，其粒度较大。
vcpupin命令就可以单独把虚拟机上的vcpu与物理机上的物理cpu进行绑定

比如vm1和vm2都有3个vcpu（core），物理机有8个cpu（8个core，假如每个core一个线程），taskset能做到把vm2的3个vcpu同时绑定到一个或者多个cpu上，但vcpupin能把vm1的每个vcpu与每个cpu进行绑定。
