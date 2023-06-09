在分区的时候，每个分区应该分多大是令人头疼的，而且随着长时间的运行，分区不管你分多大，都会被数据给占满。当遇到某个分区不够用时管理员可能甚至要备份整个系统、清除硬盘、重新对硬盘分区，然后恢复数据到新分区。

虽然现在有很多动态调整磁盘的工具可以使用，但是它并不能完全解决问题，因为某个分区可能会再次被耗尽；另外一个方面这需要重新引导系统才能实现，对于很多关键的服务器，停机是不可接受的，而且对于添加新硬盘，希望一个能跨越多个硬盘驱动器的文件系统时，分区调整程序就不能解决问题。

因此完美的解决方法应该是在零停机前提下可以自如对文件系统的大小进行调整，可以方便实现文件系统跨越不同磁盘和分区。那么我们可以通过逻辑盘卷管理（LVM，Logical Volume Manager）的方式来非常完美的实现这一功能。

解决思路：将所有可用存储汇集成池，当池中某个分区空间不够时就会从池中继续划分空间给分区，池中空间不够就可以通过加硬盘的方式来解决。

## 一、逻辑卷介绍

逻辑卷（LVM）：它是Linux环境下对磁盘分区进行管理的一种机制，它是建立在**物理存储设备**之上的一个抽象层，优点在于**灵活**管理。
**特点：**
1、动态在线扩容
2、离线裁剪
3、数据条带化
4、数据镜像

## 二、名词解释：

![lvm.png](https://www.zutuanxue.com:8000/static/media/images/2020/10/18/1602992183885.png)

- 物理卷
	物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数
- 卷组
	LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或多个物理卷组成。
- 逻辑卷
	LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。
- PE
	每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可配置的，默认为4MB。
- LE
	逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

## 三、逻辑卷使用流程

真实的物理设备---->物理卷（pv）---->卷组（vg）---->逻辑卷（lv）------>逻辑卷格式化---->挂载使用