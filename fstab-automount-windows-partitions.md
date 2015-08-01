title: Linux下通过修改fstab来自动挂载Windows 分区
id: 108
categories:
  - 学习总结
date: 2014-12-14 21:14:47
tags:
 - Linux
---

我电脑装的是Windows和Linux双系统,以前在Linux下,要打开Windows系统的C盘或D盘,总是要输入密码,很麻烦,而且麻烦了很长时间.

后来有一天浩哥看到了,说可以在Linux开机时自动挂载Windows分区,修改`/etc/fstab`这个文件,可以采用每个分区的UUID.后来校长也看到了我每次麻烦的操作,说是确实可以搞,而且他已经搞定了.我想我也得搞搞了.
<!--more-->

fstab文件位于`/etc`目录下，是一个多文件系统的信息描述文件,应用程序不能修改它,而它的维护和修改任务则需要系统管理员来完成.每个分区在fstab中表示为一行,一行有6个域(field),每个域用空格或tab键隔开.

```bash
$ cat /etc/fstab

# /etc/fstab: static file system information.
#
# Use `blkid` to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# file    mount    type    options    dump    pass             
# / was on /dev/sda6 during installation
UUID=22b1037f-6c5e-46d0-b965-44cc42313795 /             ext4       errors=remount-ro  0 1
# /home was on /dev/sda5 during installation
UUID=7c4b5af9-599b-4052-aeb1-5dbd78f4d8e8 /home         ext4        defaults          0 2
/dev/sr0                                  /media/cdrom0 udf,iso9660 user,noauto       0 0
devpts                                    /dev/pts      devpts      defaults          0 0
```

可以看到,6个域名称分别是

```bash
	file    mount    type    options    dump    pass 
```

而且Linux系统分区已经挂载好了,所以我们接下来只要添加Windows分区就可以了。
6个域详细介绍如下:

###1.file system: 
表示将要挂载的分区的块设备名称.注意这个设备也可以是远程设备,比如说是远程服务器上的某个设备.对于本地设备,该域格式可以是`/dev/cdrom`,`LABEL=<label>`,或者`UUID=<uuid>`三者之一；对于远程文件系统,格式为<host>:<dir>,如 freeshell.ustc.edu.cn:/.远端设备格式好写,对于本地设备,如何获取UUID 号和LABEL呢?我们要挂载的C盘是`/dev/sdb1`还是`/dev/sda5`呢?这个可以用`blkid`命令查看:

```bash
sudo blkid
/dev/sda1: LABEL="M-gM-3M-;M-gM-;M-^_M-dM-?M-^]M-gM-^UM-^Y" UUID="9ED61632D6160B63" TYPE="ntfs" PARTUUID="5be4a3f9-01" 
/dev/sda2: UUID="908265F98265E466" TYPE="ntfs" PARTUUID="5be4a3f9-02" 
/dev/sda3: UUID="98B6FE61B6FE3EF6" TYPE="ntfs" PARTUUID="5be4a3f9-03" 
/dev/sda5: UUID="7c4b5af9-599b-4052-aeb1-5dbd78f4d8e8" TYPE="ext4" PARTUUID="5be4a3f9-05" 
/dev/sda6: UUID="22b1037f-6c5e-46d0-b965-44cc42313795" TYPE="ext4" PARTUUID="5be4a3f9-06" 
```
我们知道,Windows的文件系统格式是ntfs(new technology file system),从上面的输出中我们可以知道,要挂载的Windows分区是`/dev/sda2`和`/dev/sda3`.因为这两个分区没有LABEL,所有就没法采用`LABEL=<label>`的方式来表示第一个域了.所以我们要挂载的两块Windows分区的第一个域可以这样写:

```bash
#C盘
/dev/sda2
#D盘
/dev/sda3
```
或者:

```bash
#C盘 
UUID=908265F98265E466 
#D盘 
UUID=98B6FE61B6FE3EF6
```

###2.mount point:

即挂载点,使用过mount命令的同学应该明白这个域是干什么的,简单来说就是将物理的存储盘在Linux系统中找一个点放置下来,相当于在Linux文件树上找一个点,将物理存储对应到这个点上.挂载在这个点后,所有对该点的操作都会写入到对应的物理存储中.在最顶上的挂载例子中,我们看到UUID=22b1037f-6c5e-46d0-b965-44cc42313795(从`blkid`命令结果可以看出,该分区是`/dev/sda6`)的物理存储挂载到了/目录(Linux系统根目录),也就是说/目录下面的所有东西都写入到该分区中(/home目录除外),同理,所有/home目录下的内容都写入到UUID=7c4b5af9-599b-4052-aeb1-5dbd78f4d8e8(从`blkid`命令结果可以看出,该分区是`/dev/sda5`)的分区中  

那么,我们要把C盘和D盘挂载到哪里呢?我是这样做的: a.先查看没有自动挂载Windows分区之前,手动挂载时,系统会把C盘和D盘挂载到哪,结果如下:`/media/wang`(wang是我的用户名),C盘被命名为908265F98265E466,D盘被命名为98B6FE61B6FE3EF6,即其相应的UUID. b.所以我想,可能是挂载到`/media`目录下的任意一个子目录下吧, 所以我将该域分别设置为`/media/c`和`/media/d`,综合前两个域,应该写成:

```bash
#C盘
/dev/sda2 /media/c
#D盘
/dev/sda3 /media/d
```
或者:

```bash
#C盘 
UUID=908265F98265E466 /media/c
#D盘 
UUID=98B6FE61B6FE3EF6 /media/d
```

###3.type 
即文件系统的格式,像Linux下常用的 ext,ext1,ext2,ext3,Windows下常用的fat16,fat32,ntfs等.可以根据blkid命令的结果来写该域.根据`blkid`的结果, 我们要挂载的C盘和D盘的文件系统格式为ntfs,所以前三个域都确定了,有如下写法:

```bash
#C盘
/dev/sda2 /media/c ntfs
#D盘
/dev/sda3 /media/d ntfs
```

或者:

```bash
#C盘 
UUID=908265F98265E466 /media/c ntfs
#D盘 
UUID=98B6FE61B6FE3EF6 /media/d ntfs
```

###4.option:

选项,该域表示挂载的时候的一些选项,主要有6个选项,每个选项用逗号隔开,下面详细说明每个选项的含义:
```bash
default:使用默认选项
noauto:当执行mount -a(即挂载全部文件系统,开机时会执行此命令)时忽略此条记录,也就是跟没写进fstab一样
user:允许特定的用户来挂载,如user=bob,则只能允许bob这个用户来挂载
owner:允许物理设备的拥有者来挂载
comment:为fstab维护程序提供一些说明
nofail:在挂载失败后,忽略此错误,继续往下执行
```

因为我们没有特殊要求,所以就选default.所以前四个域可以写成这样子:

```bash
#C盘
/dev/sda2 /media/c ntfs default
#D盘
/dev/sda3 /media/d ntfs default
```

或者:

```bash
#C盘 
UUID=908265F98265E466 /media/c ntfs default
#D盘 
UUID=98B6FE61B6FE3EF6 /media/d ntfs default
```

###5.dump

dump这个命令执行备份操作,该域为0,表示执行dump操作时忽略该分区,如果为1,则表示执行dump时也会备份该分区.因为我们没有备份的需求,所以该域设置为0,所以前五个域为:

```bash
#C盘
/dev/sda2 /media/c ntfs default 0
#D盘
/dev/sda3 /media/d ntfs default 0
```

或者:

```bash
#C盘 
UUID=908265F98265E466 /media/c ntfs default 0
#D盘 
UUID=98B6FE61B6FE3EF6 /media/d ntfs default 0
```

###6.pass:
不是passwd的pass,而是系统重启时检查分区正常与否时,该分区的检查顺序.根目录所在分区passno是1,其他分区为2.如果设置为0,则表示不检查.我们的C盘和D盘不想让Linux检查,所以设置为0.所以综合以上步骤,我们可以写出下面的完整的两条记录:

```bash
#C盘
/dev/sda2 /media/c ntfs default 0 0
#D盘
/dev/sda3 /media/d ntfs default 0 0
```

或者:

```bash
#C盘 
UUID=908265F98265E466 /media/c ntfs default 0 0
#D盘 
UUID=98B6FE61B6FE3EF6 /media/d ntfs default 0 0
```

按理来说这两种形式都可以的,将任一种形式的两条记录添加到fstab文件中,重新启动系统,下次打开Windows系统的分区时,应该就不需要输入密码了. 但正如前面提到的,使用UUID的方式更健壮些,比如有的移动硬盘或U盘,拔下来再次插入的时候`/dev/sda`的编号可能会变,但其对应的UUID不会变,所以使用UUID会省下许多麻烦,推荐使用UUID形式.
