---
title: KVM guest host hardware passthrough
date: 2019-08-10 09:08:09
tags:
- Virtualization
- KVM
---

在vmware下，物理主机的硬件映射给虚拟机称为直通, 可以很方便的通过ＵＩ进行操作, 但kvm下，需要通过技术手段实现
### kvm下的DMA（direct memory access）
<!---more --->
dma要存取的内存地址成为DMA地址（也称为bus address），dmar是DMA remapping ,是intel为支持虚拟机而设计的I/O虚拟化技术，I/O设备访问的DMA地址不再是物理内存地址，而要通过DMA remapping硬件进行转译，DMA remapping硬件会把DMA地址翻译成物理内存地址，并检查访问权限等等。负责DMA remapping操作的硬件称为IOMMU。

两种DMAR的实现方法均需要IOMMU


#### 方法一　pci-stub
开启IOMMU，之后modprobe
`modprobe pci_stub`

之后会有/sys/bus/pci/drivers/pci-stub这个模块

```SHELL
lspci -nnD 查看设备详细信息
echo "14e4 1639" > /sys/bus/pci/drivers/pci-stub/new_id
echo 0000:01:00.1 > /sys/bus/pci/devices/0000:01:00.1/driver/unbind
echo 0000:01:00.1 > /sys/bus/pci/drivers/pci-stub/bind
```
使用qemu-kvm启动虚拟机

/usr/libexec/qemu-kvm -enable-kvm -m 8192 -smp 4,maxcpus=16,sockets=16,cores=2,threads=2 -spice port=3003,disable-ticketing -vga qxl /home/vms/images/basewin7.img -device pci-assign,host=01:00.1

#### 方法二 vfio

开启IOMMU，之后加载内核模块
```SHELL
mdoprobe vfio
modprobe vfio-pci

lspci -nnD
echo 0000:05:00.0 > /sys/bus/pci/devices/0000:05.00.0/driver/unbind
echo 1000 005b > /sys/bus/pci/drivers/vfio-pci/new_id
```

使用qemu-kvm启动虚拟机

/usr/libexec/qemu-kvm -M pc-i440fx-rhel7.0.0 -enable-kvm -m 2048 -smp 2 -drive if=pflash,format=raw,file=/usr/share/edk2.git/ovmf-x64/OVMF_CODE-pure-efi.fd -drive id=disk0,if=none,format=qcow2,file=test.qcow2 -device virtio-blk-pci,drive=disk0,bootindex=0 -global PIIX4_PM.disable_s3=0 -global isa-debugcon.iobase=0x402 -debugcon file:fedora.ovmf.log -monitor stdio -device piix3-usb-uhci -device usb-tablet -netdev id=net0,type=user -device virtio-net-pci,netdev=net0,romfile= -device qxl-vga -spice port=3000,disable-ticketing -device vfio-pci,host=00:05:00.0
