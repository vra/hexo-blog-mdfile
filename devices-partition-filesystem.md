title: '设备文件,分区和文件系统辨析'
id: 176
categories:
  - 学习总结
date: 2014-12-15 14:57:56
tags:
  - Linux
---

在写上一篇博客时,我发现我没搞清楚块设备(block device),分区(partion)和文件系统(filesystem)这几个概念之间的关系,今早查了一些资料才慢慢理解了它们之间的关系,所以我想写出来,看看我能不能将一个问题描述清楚.下面我依次描述设备文件,分区和文件系统这三个概念.

<!--more-->

## 设备文件(Device file)

在类Unix操作系统中,有"[一切皆文件(everything is a file)"的思想](http://en.wikipedia.org/wiki/Everything_is_a_file),当然硬件设备也不例外.在这个思想下,打印机,CD碟片,硬盘,输入输出硬件都被视为一个文件,而这些被视为文件的物理介质就可以称为设备文件.物理介质分为字符设备和块设备,详细的含义见下.除了物理介质,Unix操作系统还有一类设备文件,叫伪设备,这三类设备文件的具体含义是:

### 字符设备(Character devices)

每次与系统传输数据时,只传输一个字符.没有缓冲区,系统直接从物理设备读取字符.常用于流设备的通信.因为没有缓存,所以只能顺序读取字符,不支持随机读取.像串口和键盘就是字符设备.

### 块设备(Block devices)

与字符设备相反,块设备每次与系统传输数据时,是以块(Block)的方式来传输的.由于以块来读取,所以需要一定读取时间,故常设有缓存区,支持随机读取.常见的块设备有硬盘,CD-ROM驱动器和闪存等.

### 伪设备(Pseudo-devices)

前面两种设备文件是物理设备,而伪设备则不是,它们通常是为操作系统提供特定的功能而存在的.常见的伪设备有:

`/dev/null`:接受和丢弃所有输入,即吞下输入,然后什么都不做.

`/dev/zero`:产生联系的NULL字符串流,用c语言表示就是"\0\0\0\0\0"

`/dev/random`:产生一个随机的字符串流

`/dev/full`:模拟一个已经装满了内容的设备

这些伪设备有什么用呢?在实际中,如果巧妙使用这些伪设备的话,可以提高工作效率,像命令

```bash
dd if=/dev/zero of=testzero count=1024 bs=1024
```

就会创建一个大小为1024的,文件名为testzero的空文件.

上面就是设备文件的大概内容.在Linux 下,设备文件都在`/dev`目录下,并且有特定的前缀,可以看看:

```bash
$ cd /dev
$ ls

audio            dvd        loop2            network_throughput  sda5      tty11  tty27  tty42  tty58    v4l         vcsa4
autofs           dvdrw      loop3            null                sda6      tty12  tty28  tty43  tty59    vboxdrv     vcsa5
block            fb0        loop4            port                sdb       tty13  tty29  tty44  tty6     vboxdrvu    vcsa6
bsg              fd         loop5            ppp                 sg0       tty14  tty3   tty45  tty60    vboxnetctl  vcsa7
btrfs-control    full       loop6            psaux               sg1       tty15  tty30  tty46  tty61    vboxusb     vfio
bus              fuse       loop7            ptmx                sg2       tty16  tty31  tty47  tty62    vcs         vga_arbiter
cdrom            gpmctl     loop-control     pts                 shm       tty17  tty32  tty48  tty63    vcs1        vhci
cdrw             hidraw0    mapper           random              snapshot  tty18  tty33  tty49  tty7     vcs2        vhost-net
char             hpet       mcelog           rfkill              snd       tty19  tty34  tty5   tty8     vcs3        video0
console          hugepages  media0           rtc                 sr0       tty2   tty35  tty50  tty9     vcs4        watchdog
core             initctl    mei              rtc0                stderr    tty20  tty36  tty51  ttyS0    vcs5        watchdog0
cpu              input      mem              rts51x0             stdin     tty21  tty37  tty52  ttyS1    vcs6        xconsole
cpu_dma_latency  kmsg       mixer            sda                 stdout    tty22  tty38  tty53  ttyS2    vcs7        zero
cuse             kvm        mixer1           sda1                tty       tty23  tty39  tty54  ttyS3    vcsa
disk             log        mqueue           sda2                tty0      tty24  tty4   tty55  uhid     vcsa1
dri              loop0      net              sda3                tty1      tty25  tty40  tty56  uinput   vcsa2
dsp              loop1      network_latency  sda4                tty10     tty26  tty41  tty57  urandom  vcsa3
```
可以看到有很多的设备文件,前面提到的`/dev/null`等伪设备也在里面.

对特定类型的设备有特定的前缀,如对硬盘,前缀是`sd`,如`sda`就是第一块硬盘.对终端设备,前缀是`tty`.

像我们的笔记本,一般只有一块硬盘,也就是只有一个块设备,我们可以将所有内容都存在这个设备上,像日志文件,`/home`下面的文件都平等的存放,谁东西多就多占点空间.

但这样有个问题,由于日志文件占地方会比较大,如果有一天,将整个设备占满之后,其他文件就没有地方放了,整个系统就没法再正常运转下去了.所有就产生了这种方案:将一块设备划分成好几个部分,比如日志文件放一个部分,`/home`文件放另一个部分,相互隔离开.如果日志文件占满了,别的空间还能正常使用,所以分区解决了上述问题.

还有如果你想装双系统,如果不分区,两个操作系统混在一起,可能会发生很多意外,所以分区显得很有必要.

## 分区(Partition)

从上面我们可以看到,分区其实就像把一个硬盘分成了好几份,就跟把一个大蛋糕切成好几块,一人一块一样.其实从前面的/dev目录下的设备文件我们可以看到,`sda`这个设备被分成了6个分区,分别是`sda1`,`sda2`,....`sda6`.就像有些动物通过撒尿来标记自己领地的边界一样,块设备也有特定的标记分区边界的文件,那就是分区表.分区表就像契约一样,规定了硬盘的前多少个空间分给分区1,后面多少空间分给分区2,等等.可以通过`fdisk`指令来查看分区详情:

```bash
$ sudo fdisk -l
Disk /dev/sda: 750.2 GB, 750156374016 bytes
255 heads, 63 sectors/track, 91201 cylinders, total 1465149168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disk identifier: 0x5be4a3f9

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1            2048      718847      358400    7  HPFS/NTFS/exFAT
/dev/sda2          718848   409602047   204441600    7  HPFS/NTFS/exFAT
/dev/sda3       409602048   819202047   204800000    7  HPFS/NTFS/exFAT
/dev/sda4      1268469758  1465147391    98338817    5  Extended
Partition 4 does not start on physical sector boundary.
/dev/sda5      1346594816  1465147391    59276288   83  Linux
/dev/sda6   *  1268469760  1346594815    39062528   83  Linux

Partition table entries are not in disk order

```
前面是硬盘的物理信息,如大小,有多少个柱面等等.后面是各个分区的开始位置,结束位置,包含多少个Blocks,系统类型等信息.

分区完成后,我们就可以在不同的分区上干不同的事情了.我把`sda2`标记为C盘,把`sda3`标记为D盘,把Linux的根目录挂载在`sda6`上,把`/home`目录挂载在`sda5`上,大家互相不再干扰,和谐共处.

## 文件系统(Filesystem)

在Windows下,我们格式化U盘的时候,会让你选择格式化为FAT16,FAT3或者NTFS等,那么这些东西又是什么东西呢?这些东西就是不同的文件系统格式.

文件系统是一种存储和组织计算机数据的方法,通过文件系统,我们可以使用简单的方式来对物理介质执行操作.比如,没有文件系统,如果我要删除一个文件,那么我就得先找到它在硬盘上的哪个扇区,哪个柱面,然后删除它.有了文件系统,我可以用图形化的界面按`Shift`+`Delete`删除.这些简便都是文件系统的功劳.如果说分区这个概念是物理上的概念的话,那么文件系统就是纯粹的逻辑上的概念了.

不同的系统支持的文件系统不同,

```bash
Windows:FAT16,FAT32,NTFS等
Linux:ext1,ext2,ext3,ext4,NTFS,ISO9660等
Mac OS X:HFS,HFS+
```

如何查看各个分区的文件系统呢?可以用`blkid`命令:

```bash
$ sudo blkid

/dev/sda1: LABEL="M-gM-3M-;M-gM-;M-^_M-dM-?M-^]M-gM-^UM-^Y" UUID="9ED61632D6160B63" TYPE="ntfs" PARTUUID="5be4a3f9-01" 
/dev/sda2: UUID="908265F98265E466" TYPE="ntfs" PARTUUID="5be4a3f9-02" 
/dev/sda3: UUID="98B6FE61B6FE3EF6" TYPE="ntfs" PARTUUID="5be4a3f9-03" 
/dev/sda5: UUID="7c4b5af9-599b-4052-aeb1-5dbd78f4d8e8" TYPE="ext4" PARTUUID="5be4a3f9-05" 
/dev/sda6: UUID="22b1037f-6c5e-46d0-b965-44cc42313795" TYPE="ext4" PARTUUID="5be4a3f9-06" 
```

可以看到,`/dev/sda1`,`/dev/sda2`和`/dev/sda3`是ntfs文件系统,`/dev/sda5`和`/dev/sda6`是ext4文件系统.(/dev/sda4去哪了呢?...)


最后用一个图来总结一下:

![filesystem](/uploads/2014/12/filesystem.png)

