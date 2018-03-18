title: Amax K40 Linux GPU服务器重装记录
date: 2017-07-21 00:19:36
tags:
 - Linux
 - GPU
 - 总结
 - Amax
 - BIOS
 - UEFI
 - VNC
 - Ubuntu
---
因为这台GPU服务器闲置了很久，经过这两天的安装，现在基本能用了。整个过程其实挺坎坷的，因此记录下此次安装过程中遇到的坑，后面好参考。服务器从原先的OpenSuse换成了Ubuntu 16.04 LTS 发行版。  
<!--more-->

### 安装系统遇到的问题
安装系统整个流程是这样的：
 1. 下载Ubuntu镜像，从科大的源上下载
 2. 制作启动U盘，采用软碟通来制作
 3. 插入U盘到机器，进入BIOS设置，选择Legacy模式，并将U盘的启动顺序放在最前面
 4. 启动从U盘启动，进入安装Ubuntu的图形界面，进行重新划分盘然后在盘上安装系统
 5. 安装好后，重启系统，拔掉U盘，进入BIOS，调整硬盘`ST2000*`的启动顺序，使之排在最前面

其中遇到两个问题。第一个问题是在上述3步骤的时候，选择Legacy还是UEFI。这里Legacy表示选择经典的BIOS启动，BIOS简单来说就是主板上一块固话的flash芯片，它在开机后会最先启动，然后按照里面写好的步骤把主要硬件挨个检查一遍，然后去硬盘找引导程序，然后把引导权交给它，然后就进系统了。UEFI则是一种新的启动方式，Legacy的启动方式要用MBR磁盘格式，UEFI启动方式要用GPT磁盘格式。**如果你安装系统时候用的是Legacy，进入系统时选择UEFI则会报错，反过来也是，因此要保持装系统和进系统时两者的一致。** **我们的机器是因为比较老旧，所以只支持Legacy，所以选择的时候选择Legacy。**  
第二个问题是在上述第4步中划分盘的时候，底下的" device for bootloader installation"这个输入框该如何选择。刚开始我们选择了划分好的某个分区，如/dev/sdd1，但是有问题，安装后总有错。后来查看网上的资料，发现都是采用没加下标的盘符，如/dev/sdd。**因此在划分盘的时候，在盘开始的位置留一些空间，然后这里选择没下标的盘符，如/dev/sdd，即可。**   

### 挂载硬盘的问题
安装好系统后，默认挂载的只有装系统时候的那块盘上的分区，像别的硬盘上的分区如果要开启自动挂载的话，需要在/etc/fstab里面写入记录。  
fstab的记录的格式是这样的：
```bash
# format <file system> <mount point>   <type>  <options>       <dump>  <pass>, for example,
/dev/sdc /data2 ext4  defaults 0 0
```
其中细节可以参考我写的[这篇博客](https://vra.github.io/2014/12/14/fstab-automount-windows-partitions/)。其中type这个参数不好确定，需要用`blkid`命令来查看，如下面两种格式：
```bash
$ sudo blkid
/dev/sda1: UUID="d0dbd540-305b-493d-b42b-f9d92f3388d7" TYPE="ext4"
/dev/sda2: UUID="9a9946ab-43b4-454a-9993-b59fb4860139" TYPE="ext4"
/dev/sda3: UUID="479416ea-cbf0-42ef-9f19-827f0524128a" TYPE="swap"
/dev/sdb1: UUID="89e6c107-62bd-4e7a-807d-5b5024f272b5" TYPE="ext4"
/dev/sdc: UUID="117d0fa5-cffb-45ff-8758-9dc23063bcf8" TYPE="ext2"
/dev/sdd: UUID="887d5d9a-fbb3-4036-ad97-11b072cff277" TYPE="ext4"
/dev/sde: UUID="61ae41de-1fca-4d4e-bdb3-e45f8d9777ec" TYPE="ext3"
/dev/sdf: UUID="5d23d7a7-b572-4973-846c-7cf23d1a915f" TYPE="ext4"
/dev/sdg: UUID="088ccdbd-ad9d-4bf1-994c-7bd11b714c7a" TYPE="ext4"
$ sudo blkid -o value -s TYPE /dev/sda1
ext4
```
知道这个参数之后，就修改/etc/fstab，增加要挂载的硬盘分区，然后重启即可。**主要挂载如果参数有错的话，开机启动会报错，进入安全模式，须将其修改后再启动。所以备份原先的fstab是很有必要的。**  
### Caffe编译过程中HDF5库路径的问题
在Ubuntu 16.04中，HDF5文件的lib目录不在系统默认的/usr/local/lib或/usr/lib目录下，而是在`/usr/lib/x86_64-linux-gnu/hdf5/serial`目录下，所以在Caffe的Makefile.config中的`LIBRARY_DIRS`那行后面增加` /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial`。同理，其include目录也不在标准目录，而是在`/usr/include/hdf5/serial`目录，因此也需要将其加入到`INCLUDE_DIRS`这一行，因此这两行的最终结果如下：
```bash
93 # Whatever else you find you need goes here.
94 INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
95 LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
```
修改后再重新编译Caffe即可。

### cuDNN6.0和tensorflow
在尝试用pip安装tensorflow的时候，安装后执行`import tensorflow as tf`的时候，报错，说是找不到cuddnn.so.5，可是pip安装的是用cuDNN5.\*编译的二进制文件，也即当前pypi仓库里面是用的cuDNN5.0。为了能用cuDNN 6.0加速性能，我决定自己编译Tensorflow。之前在Ubuntu 14.04上编译过一次，但由于包太老，bazel安装出现问题。这次编译没错，安装[官方文档](https://www.tensorflow.org/install/install_sources)来操作即可。  

### VNC 显示不全的问题
这个问题现在还没解决，但是基本可以使用了，只不过在VNC界面打开Terminal的时候会报错，而打开Xterm却没问题，网上找了些资料也没好的解决方法。因此这里先把目前的操作记录下来吧。  
#### 使用Gnome界面
需要安装一下Gnome图像界面的包：
```bash
sudo apt-get install --no-install-recommends ubuntu-desktop gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```
然后编辑~/.vnc/xstartup文件，修改为如下：
```bash
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus -n &
gnome-terminal &
```
然后关掉之前的VNC连接：
```bash
vncserver -kill :1
```
重新设置新的连接：
```bash
vncserver -geometry 1840x1020 :1
```
然后再客户端连接服务器，查看是否有问题

#### 安装xfce桌面
这时候需要安装如下的桌面显示包：
```bash
sudo apt install xfce4 xfce4-goodies
```
然后将~/.vnc/xstartup修改为如下：
```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```
然后执行和上面同样的kill掉旧连接，建立新连接。


### 设置静态IP
服务器重装后，IP地址改变了，但是我们还是希望大家使用熟悉的IP地址，所以需要设置一个静态IP地址。这里实现的步骤如下：  
先是执行`sudo ifconfig`来查看以太网网卡的名称，因为后面要用到：
```bash
$ sudo ifconfig
enp4s0f0  Link encap:Ethernet  HWaddr 0c:c4:7a:32:6e:42
          inet addr:192.168.134.200  Bcast:192.168.104.255  Mask:255.255.255.0
          inet6 addr: 2101:da0:d800:1454:725e:fb01:5ef8:16a5/64 Scope:Global
          inet6 addr: 2101:da0:d800:1454:2234:4e15:e803:59b9/64 Scope:Global
          inet6 addr: fe80::ec4:7aff:fe34:2e42/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1764153 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1086158 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2514858243 (2.5 GB)  TX bytes:398311453 (398.3 MB)
          Memory:c7020000-c703ffff

enp4s0f1  Link encap:Ethernet  HWaddr 0c:c4:7a:04:6e:43
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Memory:c7000000-c701ffff

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:1836 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1836 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:246069 (246.0 KB)  TX bytes:246069 (246.0 KB)
```
可以看到其中有3个网络接口，第三个`lo`应该是本地loop接口，不是网卡，前两个，可以看到第一个数据流量大，而第二个数据流量都是0，所以确定网卡名称为enp4s0f0。然后编辑`/etc/network/interfaces`文件，内容如下：
```bash
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto enp4s0f0
iface enp4s0f0 inet static
address 192.168.134.234
netmask 255.255.255.0
gateway 192.168.134.254
dns-nameservers 202.38.64.1 202.141.176.93
```
其中`enp4s0f0`根据你的上条命令的结果自行调整。  
修改完后，执行下面的命令重启网络服务：
```bash
sudo systemctl restart networking.service
```
之后重启系统，就会发现IP已经变为你期望的静态IP。  

### MatLab 安装
MatLab安装现在还没完成，但已经有一些地方值得记录下来。  
先是从学校正版软件中心下载ISO文件，然后解压文件，放在某个目录下，修改其中的installer_inputer,主要修改下面这些地方：
```bash
destinationFolder=/usr/local/R2017a
fileInstallationKey=xxxx-xxxx-...
agreeToLicense=yes
outputFile=/tmp/mathworks_yunfeng.log
mode=silent
licensePath=/data/public/2017a/network.lic
```
**注意最后一个参数，注释的例子里面都是写的`.dat`文件，很误导人，其实在这里写成到`network.lic`的绝对路径即可。**  
然后退出MatLab目录，在别的目录执行：
```bash
sudo ./data/public/R2017a_glnxa64_dvd1/install -inputFile /data/public/R2017a_glnxa64_dvd1/installer_input.txt
```
这里有个问题，2017a是分2个ISO文件的，一个文件执行完后，另一个文件怎么插入呢？网上有ISO文件编译时的说明，但是想想貌似不可行，以后再试试。  
