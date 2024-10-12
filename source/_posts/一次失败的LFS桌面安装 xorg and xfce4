---
title: 一次失败的LFS桌面安装 xorg and xfce4
layour: post
uuid: fasw2d-aLFS-11ed-fasfw-wqr2fg
tags: [LFS]
categories: LFS
date: 2024-10-12 18:07:08
---

自从安装好LFS后就一直放着没有动，但是ddl马上就要到了，又要求整一个桌面环境，拜托，安装好LFS就已经要了我大半条命了欸！

感觉有编译恐惧症了哈哈。

看了一下官方的文档，安装一个Gnome居然要死多死多的依赖，我的天哪！感觉人要挂掉了。

唉，不过还是要继续。在LFS安装后，我才发现我在安装的过程中并没有配置网络，研究了一下配置网络，花费两小时。后来好不容易有了网络后，才发现刚刚安装好的LFS系统并没有安装wget，curl之类的网络工具，我去，这是要逼死我啊！

安装一个wget要依赖一个别的，然后又依赖下去...人都要挂了。最重要的是LFS没有地方去上传文件。也没有ssh，scp之类的工具，都要重新编译安装。这里又研究了一个小时去找方法传输文件...

virtual box的共享文件夹，因为无法安装增强工具，用不了...

然后我发现我有u盘啊！我可以用U盘来传文件。到了这里又发现了一个问题，那就是U盘的格式是NTFS，LFS不支持啊！！！用diskgenius把U盘格式化成ext4，然后挂载到另外一个Linux虚拟机中。再挂载到LFS中，终于可以传文件了。不容易啊不容易啊！！！

最先安装的必须要是SSH，默认的终端太难受了，还不能复制粘贴！！！

有了ssh后就可以连上了，从U盘中cp文件到LFS中再进行安装，这过程又是好长时间。。

到这一步我已经用掉了80%的硬盘容量，总共就30G，不过当时我在Debian上分配的是50G，我还有20G的空间，不过这个空间是在Debian上的，LFS是没有的。因为已经把引导改为LFS了，所以Debian的空间就能直接删掉然后重新分配给LFS。这一步需要进入到一个Live系统，因为我们无法直接在主分区上进行操作。这里不给过程了。自己动手手动操作一下吧！

过程..

![](https://img.164314.xyz/2024/10/11f560b93ade3429711a944c8cf7e4f2.png)

- 扩容前

![](https://img.164314.xyz/2024/10/379fa820ac5d25cb33f41ba3056d9f9e.png)

- 扩容后

![](https://img.164314.xyz/2024/10/48352148929ba15f54cb1ba16d0a8e71.png)

因为我修改了swap分区的名字，所以需要修改fstab文件，不然会报错。原来是`/dev/sda5`，现在是`/dev/sda1`。修改一下/etc/fstab文件里的/sda5为/sda1就好了。

# 安装xfce4

首先到这个网站下先把所有的依赖都下来过来，这样子能省一点时间。[BLFS](https://www.linuxfromscratch.org/blfs/view/stable/xfce/xfce.html)

## 1. libxfce4util

在进行安装之前，这个包有个依赖我们并没有安装，那就是`glib`，我们先安装这个包。
###  1.1 glib

在这个网站上下载所需要的包：[glib](https://www.linuxfromscratch.org/blfs/view/stable/general/glib2.html)

记得要下载主体的包和Recommended包。也就是Glib和Gobjetct

然后就是解压，编译，安装。

```bash
mkdir build &&
cd    build &&

meson setup ..                  \
      --prefix=/usr             \
      --buildtype=release       \
      -D introspection=disabled \
      -D man-pages=enabled      &&
ninja
ninja install
```

这其中又遇到了这个依赖没有，那个依赖没有，然后又去下载，又去编译，又去安装。我遇到的有`docutils`,`rst2man`, `pcre21`等等。这其中大家伙可以自己去看看官方文档，我就不一一列举了。

出现这个页面代表编译成功

![](https://img.164314.xyz/2024/10/f1e0fc710ce29d8f8ddcc3831c4b1331.png)

然后这个是安装成功的页面

![](https://img.164314.xyz/2024/10/b5d7c39dc91973205b03ed4fe43c90da.png)

到这里应该就没有依赖问题了，接下来将下载过来的文件解压，然后进入到文件夹中，执行以下命令：

这个压缩包是tar.bz2 格式，用 `tar -jxvf` 解压。

```bash
./configure --prefix=/usr &&
make

make install
```

![](https://img.164314.xyz/2024/10/ed1764aad20e9e4755ca222ad95a41a1.png)

## 2. Xfconf

这个包依赖前一个包，所以一定要有了前一个后在进行这个

```bash 
./configure --prefix=/usr &&
make

make install
```

![](https://img.164314.xyz/2024/10/0961a45b47077ae8c2521c027cc2f0ef.png)

## 3. libxfce4ui

这个包的依赖也是老多了...

还是一样的步骤，少什么就装什么。都安装好了就可以运行下方的命令了。

-----

写道这里就不想写了。因为在这其中遇到了好多依赖没有安装上，这个少那个，那个少这个，太多依赖要安装了。真要写上也没必要，毕竟都在手册里了。我就说几个我遇到的难题吧。

在我进行编译gobject-introspection的时候,一直遇到一个ldd问题，总是编译不成功，我问了chatGPT和Claude还有另外的其他的AI，但都没有解决。最后我粘贴到谷歌上，就像是十年前我遇到问题直接把问题粘贴上去。然后我就发现了这个帖子

https://www.linuxquestions.org/questions/linux-from-scratch-13/cant-build-gobject-introspection-4175661875/

这里的人的问题和我遇见的是一模一样的。也是无法编译gobject-introspection。然后我就看到了这个回答

![](https://img.164314.xyz/2024/10/597a19148f5ef1504684994fed02461b.png)

当我尝试按照他说的去做的时候，我发现，编译成功了！！！这狗问题困扰我一天有余！！！互联网上的力量真的是太强大了！！！

另外当我安装完Xorg，准备启动startx，一直在报错。也不知道为什么，看见有人说安装个[xf86-video-vmware 13.4.0-3
](https://archlinux.org/packages/extra/x86_64/xf86-video-vmware/) 就好了，因为我是虚拟机。我也不知道有没有，但是先安装上。真有用的话那可真是幸运

还有就是内核问题，内核中还需要启用一下桌面环境的支持，我想着当初我编译内核的时候没有启用这个。重新去编译然后安装应该就好了。虚拟机备份快照，真机备份硬盘。毕竟内核编译是最危险的。

这个是配置

![](https://img.164314.xyz/2024/10/7b624f87351ad0919d38913d20174a93.png)

然后这里是过程

![](https://img.164314.xyz/2024/10/cdbebaf8393a257b8888306862f7d716.png)

![](https://img.164314.xyz/2024/10/42f463fdf923f9c6383191ce20cabb38.png)

![](https://img.164314.xyz/2024/10/b0a261031df0a751e20b3386fc58d73b.png)


-----

10.12.2024

从10.7.2024开始搞，一直在安装Xorg环境。但是当我使用startx启动桌面的时候就是启动不了。报错 no screen found 之类的。我查了好多资料。但还是解决不了。我用的是virtual box虚拟机。可能以后用真机的时候会翻出来这个来看一看当初的我吧。

发出来给自己看。这文也是在七号就写好大半的，后面就一直在重新安装xorg，安装xorg。。

太可惜了。。。
