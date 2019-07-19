---
layout: post
title: "host cpu comprehension"
description: ""
category: 
tags: []
---

## Linux 物理机 ##

>查看Linux系统中物理与逻辑cpu的相关信息

>Linux 下/proc/cpuinfo文件会显示cpu的信息

>逻辑CPU个数是指cat /proc/cpuinfo 所显示的processor的个数

>cat /proc/cpuinfo | grep “processor” | wc -l

>物理CPU个数，是指physical id（的值）的数量

>cat /proc/cpuinfo | grep “physical id” | sort | uniq | wc -l

>每个物理CPU中Core的个数：每个相同的physical id都有其对应的core id。如core id分别为1、2、3、4，则表示是Quad-Core CPU，若core id分别是1、2，则表示是Dual-Core。

>cat /proc/cpuinfo | grep “cpu cores” | wc -l

>逻辑CPU：每个物理CPU中逻辑CPU(可能是core, threads或both)的个数：

>cat /proc/cpuinfo | grep “siblings”

>它既可能是cores的个数，也可能是core的倍数。当它和core的个数相等时，表示每一个core就是一个逻辑CPU，若它时core的2倍时，表示每个core又enable了超线程（Hyper-Thread）。

>比如：一个双核的启用了超线程的物理cpu，其core id分别为1、2，但是sibling是4，也就是如果有两个逻辑CPU具有相同的”core id”，那么超线程是打开的。

## Xenserver ##
**XenServer中Socket、Core、以及超线程后的核心之间在XenServer中CPU的排序关系**

![enter image description here][1]

[1]: https://lh4.googleusercontent.com/-mKqIVhoVnx4/Uyr2yJi9h_I/AAAAAAAAAHs/4_ekJs8SipY/w626-h162-no/CPU.png
