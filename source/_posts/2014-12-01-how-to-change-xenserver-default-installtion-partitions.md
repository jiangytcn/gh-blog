---
title: "How To Change Xenserver Default Installtion Partitions"
description: ""
category: 
tags: [Xenserver]
---

##  修改root磁盘大小

     开机选择F2高级模式，输入shell，
     vi /opt/xensource/installer/constants.py
     DOM0_MEM=752为DOM0_MEM=2940, 修改root_size=4096为 root_size=10240
     输入EXIT

## 修改内存
    XenServerDomain0默认使用752MB内存，由于每启动一台虚拟机，Domain0中就会启动一个Qemu-DM的进程，占用大约6M的内存空间，
	因此在虚拟机数量较多的情况下，我们需要增大Domain0内存以便支持更多的虚拟机运行。
	由于Domain0是32位操作系统，故支持的最大内存量为4GB。
	更改Domain0内存的方法参考CTX124806-XenServerSingleServerScalabilitywithXenDesktop提到的例子，
	更改/boot/exlinux.conf下包含dom0_mem=2940M的Xen命令行。

<!-- more -->


### 为了修改先前的设置，完成下面的步骤：
1. 通过XenCenter的控制口或者SSH方式以root身份登录到Domain0。
2. 确保备份一份原始的/boot/extlinux.conf文件。以免未来的修改导致XenServer不能启动。
3. 用vi打开/boot/extlinux.conf。
4. 下面的改动仅添加到labelxe和labelxe-serial部分。
	###进行下面的改动：
		labelxe#XenServerkernelmboot.c32append/boot/xen.gzdom0_mem=752Mlowmem_emergency_pool=1Mcrashkernel=64M@32Mconsole=com1vga=mode-0x0311—/boot/vmlinuz-2.6-xenroot=LABEL=root-ecpmuteuroxencons=hvcconsole=hvc0console=tty0quietvga=785splash—/boot/initrd-2.6-xen.img
	###改变为:
		labelxe#XenServerkernelmboot.c32append/boot/xen.gzdom0_mem=2940Mlowmem_emergency_pool=1Mcrashkernel=64M@32Mconsole=com1vga=mode-0x0311—/boot/vmlinuz-2.6-xenroot=LABEL=root-ecpmuteuroxencons=hvcconsole=hvc0console=tty0quietvga=785splash—/boot/initrd-2.6-xen.img


调整Dom0内存设置dom0_mem=2940M是为了分配给dom0更多的内存，这意味着它可以更好地处理更多数量的虚拟机。
在改变了这个设置并重启以确保新配置的dom0内存大小生效
