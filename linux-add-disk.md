title: Linux服务器增加硬盘操作记录
date: 2017-02-24 22:19:15
tags:
 - Linux
 - RAID
 - GPU
 - Amax
 - ext4
---
###概述
最近我们实验室的GPU服务器数据空间不够用了，老师让我联系公司来增加硬盘。我这里记录一下对Amax公司生产的GPU服务器增加硬盘的步骤。
<!--more-->
机器的参数：
 1. 操作系统：Ubuntu 14.04
 2. 显卡： Nvidia  Tesla K80
 3. 机器厂商： Amax
 4. 是否有RAID: 有

###配置RAID
RAID(Redundant Array of Independent Disks)，即独立硬盘冗余阵列，是一种管理较大空间硬盘阵列的方法，常见的RAID方式到RAID 0-RAID 6，简单的来讲可以这样理解：
 1. RAID 0: 数据不做备份操作，每块盘都可以存储数据
 2. RAID 1: 将一半的磁盘作为镜像磁盘，空间利用率只有50%，但是允许有一半的磁盘坏掉（坏掉后备份盘可以继续使用）
 3. RAID 5: 使用1块盘作为备份，别的盘可以正常存取数据
关于RAID 各种方式的细节，可以看[这里](https://www.zhihu.com/question/20131784)。
因为我们想让数据盘尽可能被充分地利用，所以我们采用RAID 0。  
将硬盘插入到插槽后，开机启动服务器，就可以进入RAID的设置。在设置页面中，选择“Configuration Wizard”开始设置。具体的设置内容可以参看[这篇博客](https://siliconmechanics.zendesk.com/hc/en-us/articles/205066349-Creating-a-RAID-0-1-5-or-6-through-the-LSI-Web-BIOS)。  

###对硬盘分区
设置好RAID后，重启进入系统，查看新加的硬盘。  
通过`sudo fdisk -l`可以查看所有连接的系统的硬盘，而`df -h`则只显示挂载到系统的硬盘，所以查看前者中有而后者中不存在的硬盘，比如`/dev/sdf`，就是我们新加的硬盘。  
找到新加的硬盘后，我们采用`sudo fdisk /dev/sdf`命令来对/dev/sdf硬盘创建分区表，输入该命令后，结果如下：
```bash
~ ᐅ sudo fdisk /dev/sdf
Device contains neither a valid DOS partition table， nor Sun， SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x083d94fb.
Changes will remain in memory only， until you decide to write them.
After that， of course， the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

The device presents a logical sector size that is smaller than
the physical sector size. Aligning to a physical sector (or optimal
I/O) size boundary is recommended， or performance may be impacted.

命令(输入 m 获取帮助)： 
```
根据提示，我们输入m，得到如下反馈：
```bash
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

命令(输入 m 获取帮助)： 
```
可以看到列出了所有可能的选项。我们这里输入n，得到输出：
```bash
Partition type:
   p   primary (0 primary， 0 extended， 4 free)
   e   extended
Select (default p):   
```
因为我们在新加的硬盘上只创建一个分区，而且新加的盘用作数据盘，不会作为启动分区，所以选Primary 分区和extended分区都没关系。从这里开始，我们所有的操作都可以选择默认，即每次都是按`Enter`键到下一步。到所有设置到完成后，fdisk命令会创建分区，大概需要等1分钟。  


###格式化硬盘
创建好分区表后，需要格式化硬盘，将Linux的文件系统应用到硬盘上，硬盘才能存储数据。格式化硬盘采用的是`mkfs`命令。  
目前Linux常用的文件格式是ext3和ext4，其中ext4是ext3的后续版本，对后者进行了一些改进，例如最大文件变成16TB、最大子目录数高达64000个等。具体的改进请参考[这里](https://en.wikipedia.org/wiki/Ext4)。使用mkfs命令时，可以使用-t 选项制定文件格式。不指定默认的文件格式是ext2。  
所以我们这里的命令是：
```bash
sudo mkfs -t ext4 /dev/sdf
```
对于弹出的问题，选择`y`即可，可以看到会写入inode数等操作，进行格式化。  


###挂载硬盘
硬盘格式化后，只要挂载到系统就可以正常使用了。接下来的操作就跟插硬盘或U盘到服务器上时的操作一样，先创建一个目录，然后将硬盘挂载到该目录，然后就可以在挂载后的目录里面写入或读出文件了，所有操作都在会在硬盘上进行。具体命令如下：
```bash
sudo mkdir /data5
sudo mount /dev/sdf /data5
sudo chmod -R 777 /data5
```
注意最后一步需要修改文件夹的权限，否则服务器上的其他用户没有读写的权限。  

###将挂载信息写入到fstab
如果只执行了挂载操作而不将硬盘的挂载操作写入到`/etc/fstab`中，则下次重启的时候，需要手动挂载，而用户对于/data5目录是无法进行读写操作的。所以接下来我们需要将挂载操作命令写入到`/etc/fstab`文件中。  
fstab命令的写法有两种，一种是采用UUID，如：
```bash
UUID=8aeec127-62bd-4e7a-2020-5a5024f27a22 /data1 ext4 defaults 0 0
```
这种格式，其中硬盘对应的UUID号可以通过命令`sudo file -sL /dev/sdf`得到。关于fstab命令后面参数的含义，请参见我的[另一篇博客](http://goingthink.wang/2014/12/14/fstab-automount-windows-partitions/)。  
另外一种格式就是用`/dev/sdf`来代替UUID，即一条记录如下：
```bash
/dev/sdf /data5 ext4  defaults 0 0
```
添加该记录到`/etc/fstab`文件后，下次重启，硬盘也会自动挂载。  
至此，我们的任务就算大功告成了，希望对你有所帮助。

