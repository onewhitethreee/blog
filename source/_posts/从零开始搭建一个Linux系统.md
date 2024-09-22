---
title: 从零开始搭建一个Linux系统
layour: post
uuid: fasw2d-arr-11ed-faswr-ttwwq
tags: [LFS, Linux, LinuxFromScratch, LinuxFromScratch中文版]
categories: LeetCode
index_img: https://img.164314.xyz/2024/09/d3734252c52ebdcdbadd5f586f99acb2.jpg
date: 2024-09-22 15:00:00
---

https://www.linuxfromscratch.org/ 

**本文所有的内容都来自于上方的网站中，我只是手动的进行了一次操作**

---

一直以来听到过很多发行版的Linux，像是Ubuntu、CentOS、Debian等等，但是对于Linux的内核和系统的构建过程却是一无所知。这次就来从零开始搭建一个Linux系统，了解一下Linux系统的构建过程。

# 1. 先决条件

- 一台Linux系统的主机（可以是VMware，也可以是主机）
- 有四个以上 CPU 核⼼和⾄少 8 GB 存储
  



# 2. 开始搭建

## 2.1. 安装Debian

``预计时间：10分钟``

首先我们需要安装一个Debian系统，这里我们使用VMware来安装Debian系统。首先我们需要下载Debian的ISO文件，然后在VMware中新建一个虚拟机，选择Debian的ISO文件进行安装。这里给出一张正在使用VMware安装Debian的截图。一路Continue，有需要设置账户名和root/user的密码需要设置一
下，直到安装完成。

在分区的时候我们留下一个空的分区，用于后面的LFS系统。可以按照我的截图来分区或者你自己选择。

![](https://img.164314.xyz/2024/09/e0cf9a962e5309c1df7d11233a57ad66.png)

PS: 需要注意的是在最后一步，不能直接选择Continue。需要如图所选再点击Continue。

![](https://img.164314.xyz/2024/09/7e7bdc987d25ba491555d178ee8a62af.png)
![](https://img.164314.xyz/2024/09/4970005fe87ae9f1fb13e3e39f9b5ec8.png)
然后系统会重启，等待一分钟即可进入系统（视性能而定）。

![](https://img.164314.xyz/2024/09/85763a4e2a42446bc2ce5399af98ab43.png)

## 进入Debian

## 1.1. 安装必要的软件(最低要求)

下列软件必须是要在宿主系统下进行安装的。这些软件是构建LFS系统的必要软件。

```
    •Bash-3.2 (/bin/sh 必须是到 bash 的符号链接或硬连接)
    •Binutils-2.13.1 (⽐ 2.43.1 更新的版本未经测试，不推荐使⽤)
    •Bison-2.7 (/usr/bin/yacc 必须是到 bison 的链接，或者是⼀个执⾏ bison 的⼩脚本)
    •Coreutils-8.1
    •Diffutils-2.8.1
    •Findutils-4.2.31
    •Gawk-4.0.1 (/usr/bin/awk 必须是到 gawk 的链接)
    •GCC-5.2，包括 C++ 编译器 g++ (⽐ 14.2.0 更新的版本未经测试，不推荐使⽤)。C 和 C++ 标准库 (包
    括头⽂件) 也必须可⽤，这样 C++ 编译器才能构建宿主环境的程序
    •Grep-2.5.1a
    •Gzip-1.3.12
    •Linux Kernel-4.19
    •M4-1.4.10
    •Make-4.0
    •Patch-2.5.4
    •Perl-5.8.8
    •Python-3.4
    •Sed-4.1.5
    •Tar-1.22
    •Texinfo-5.0
    •Xz-5.0.0
```

打开终端，并且输入nano version-check.sh, 然后粘贴以下代码即可检查环境，需要注意必须已经安装了上方所列出的所有软件：

```
#!/bin/bash
# A script to list version numbers of critical development tools
# If you have tools installed in other directories, adjust PATH here AND
# in ~lfs/.bashrc (section 4.4) as well.
LC_ALL=C
PATH=/usr/bin:/bin
bail() { echo "FATAL: $1"; exit 1; }
grep --version > /dev/null 2> /dev/null || bail "grep does not work"
sed '' /dev/null || bail "sed does not work"
sort /dev/null || bail "sort does not work"
ver_check()
{
if ! type -p $2 &>/dev/null
then
echo "ERROR: Cannot find $2 ($1)"; return 1;
fi
v=$($2 --version 2>&1 | grep -E -o '[0-9]+\.[0-9\.]+[a-z]*' | head -n1)
if printf '%s\n' $3 $v | sort --version-sort --check &>/dev/null
then
printf "OK: %-9s %-6s >= $3\n" "$1" "$v"; return 0;
else
printf "ERROR: %-9s is TOO OLD ($3 or later required)\n" "$1";
return 1;
fi
}
ver_kernel()
{
kver=$(uname -r | grep -E -o '^[0-9\.]+')
if printf '%s\n' $1 $kver | sort --version-sort --check &>/dev/null
then
printf "OK: Linux Kernel $kver >= $1\n"; return 0;
else
printf "ERROR: Linux Kernel ($kver) is TOO OLD ($1 or later required)\n" "$kver";
return 1;
fi
}
# Coreutils first because --version-sort needs Coreutils >= 7.0
ver_check Coreutils sort 8.1 || bail "Coreutils too old, stop"
ver_check Bash bash 3.2
ver_check Binutils ld 2.13.1
ver_check Bison bison 2.7
ver_check Diffutils diff 2.8.1
ver_check Findutils find 4.2.31
ver_check Gawk gawk 4.0.1
ver_check GCC gcc 5.2
ver_check "GCC (C++)" g++ 5.2
ver_check Grep grep 2.5.1a
ver_check Gzip gzip 1.3.12
ver_check M4 m4 1.4.10
ver_check Make make 4.0
ver_check Patch patch 2.5.4
ver_check Perl perl 5.8.8
ver_check Python python3 3.4
ver_check Sed sed 4.1.5
ver_check Tar tar 1.22
ver_check Texinfo texi2any 5.0
ver_check Xz xz 5.0.0
ver_kernel 4.19
if mount | grep -q 'devpts on /dev/pts' && [ -e /dev/ptmx ]
then echo "OK: Linux Kernel supports UNIX 98 PTY";
else echo "ERROR: Linux Kernel does NOT support UNIX 98 PTY"; fi
alias_check() {
if $1 --version 2>&1 | grep -qi $2
then printf "OK: %-4s is $2\n" "$1";
else printf "ERROR: %-4s is NOT $2\n" "$1"; fi
}
echo "Aliases:"
alias_check awk GNU
alias_check yacc Bison
alias_check sh Bash
echo "Compiler check:"
if printf "int main(){}" | g++ -x c++ -
then echo "OK: g++ works";
else echo "ERROR: g++ does NOT work"; fi
rm -f a.out
if [ "$(nproc)" = "" ]; then
echo "ERROR: nproc is not available or it produces empty output"
else
echo "OK: nproc reports $(nproc) logical cores are available"
fi
```



然后按Ctrl+X，然后按Y，然后按Enter，保存文件。然后输入chmod +x version-check.sh，然后输入./version-check.sh，然后等待一会，就可以看到你的系统是否满足要求了。


![](https://img.164314.xyz/2024/09/17246927482f5abb82b12c6ae233a692.png)

在截图上可以看见输出了两个error。
`Error: sh is not Bash`
`Error: g++ does NOT work`
这两个error很好解决，第一个需要修改一下链接，第二个需要安装g++。
### 修改链接

在我的虚拟机中，输入`ls -l /usr/bin/sh`，可以看见sh指向的是dash，而不是bash。所以我们需要修改一下。输入`sudo ln -sf /usr/bin/bash /usr/bin/sh`

![](https://img.164314.xyz/2024/09/2a10f8936500b6b27bf178c7def496b1.png)


### 安装g++

输入`sudo apt install g++`

再次运行检查脚本，可以看见已经没有error了。

![](https://img.164314.xyz/2024/09/56e6d098d6f7bcf9b19c7f50f719cc8d.png)

**在进行下一步前必须要确定你的系统满足上方的所有要求。如果有不满足的，需要安装相应的软件。**

## 2.1 分区

终端输入`sydi oarted /dev/sda print free`, 可以看见我还有一个空的分区没有分配，这里的空的空间是在安装系统时刻意去设置的。

记下起始位置和结束位置。接下来我们就要对这个分区进行分区。

![](https://img.164314.xyz/2024/09/c9ff48d3715e10dee6443433a520ac41.png)

`sudo parted /dev/sda`进入parted，然后输入`mkpart primary ext4 16.GB 100%`，这里的16GB是起始位置，然后使用100%。最后输入`quit`退出parted。
![](https://img.164314.xyz/2024/09/eb88e983ea2e42eda3986582332b8ea6.png)
出现了一个information。先不用管，后面会解决。

接下来运行
```
export LFS=/mnt/lfs
mkdir -pv $LFS
mount -v -t ext4 /dev/sda3 $LFS
```
如果一切正常那么你会看见新的分区已经挂在在了/mnt/lfs下。

![](https://img.164314.xyz/2024/09/cc3e20aa499d3a939093dbbef0f56857.png)


**需要的操作用户为root**

# 3. 下载需要的包

``预计时间：20分钟``

下载的依赖/软件需要放在一个容易找到的地方，这里我们选择放在/mnt/lfs/sources下。运行 `mkdir -v $LFS/sources`，然后运行 `chmod -v a+wt $LFS/sources`。然后我们就可以下载依赖了。
![](https://img.164314.xyz/2024/09/95f3e934770bc63d5039383ca083c7a2.png)

这里使用wget来下载。按照 `wget --input-file=wget-list-sysv --continue --directory-prefix=$LFS/sources`来进行所有的下载。

不过首先要创建一个wget-list-sysv文件，然后把所有的下载链接放进去。这里我就不一一列举了，可以参考[这里](http://www.linuxfromscratch.org/lfs/view/stable/chapter03/packages.html)。

[这个是上面两个链接的整合，下载这个比较好](https://www.linuxfromscratch.org/lfs/downloads/stable/wget-list)


PS: 可以使用vscode来获取所有的下载链接，正则匹配所有链接，CTRL+SHIFT+L选中所有的链接，粘贴下载即可。

下载完毕的截图
![](https://img.164314.xyz/2024/09/2ae49d40eb8d33dd05d4eed18e82bd77.png)

`cd $LFS/sources`，然后就可以看到下载完毕后的文件了。
![](https://img.164314.xyz/2024/09/582fc2ae2c8d3183ecb1c0800bab3e26.png)

不要忘了还要下载一些补丁，可以参考[这里](https://www.linuxfromscratch.org/lfs/view/stable/chapter03/patches.html)。

最后还得要运行一下`chown root:root $LFS/sources/*`

## 3.1 最后准备工作

在终端运行

```
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
for i in bin lib sbin; do
ln -sv usr/$i $LFS/$i
done
case $(uname -m) in
x86_64) mkdir -pv $LFS/lib64 ;;
esac

mkdir -pv $LFS/tools
```

![](https://img.164314.xyz/2024/09/6e2624d7c8c261272b478f70bc6b2708.png)

## 3.2 添加LFS用户

```
groupadd lfs

useradd -s /bin/bash -g lfs -m -k /dev/null lfs

passwd lfs

chown -v lfs $LFS/{usr{,/*},lib,var,etc,bin,sbin,tools}
case $(uname -m) in
x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

如果出现找不到命令的错误，可以使用 `/usr/sbin/groupadd lfs`和 `/usr/sbin/useradd -s /bin/bash -g lfs -m -k /dev/null lfs`亦或者cd到目录手动运行。上面四条命令是为了创建一个lfs用户，然后把lfs用户的权限赋予给$LFS目录。

![](https://img.164314.xyz/2024/09/e3c2ea7275ea87f10219f0e994a1ded2.png)

## 3.3. 切换到lfs用户

```
su - lfs
```

![](https://img.164314.xyz/2024/09/f65e6885ee198e006460eb41dbf65dfd.png)

## 3.3 配置lfs用户环境

``预计时间：5分钟``

分别运行以下命令

```
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

```
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

![](https://img.164314.xyz/2024/09/b85c1083a66b9b3e56b1a0fe8665bdeb.png)

防止对root用户的影响，我们需要运行

`[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE`


**最后，不要忘记 `source ~/.bash_profile`。终端输入。**

## 3.4 make -jx(可选)

提升编译速度

因为我这里使用的是虚拟机，性能不会有真机那么强大。

但是如果你的电脑性能足够强大，可以指定make -jx，这样可以加快编译速度。x为一个数字，代表你的CPU核心数。

![](https://img.164314.xyz/2024/09/44df2030340d156f031c3f51095a6ec3.png)

# 4. 编译

``预计时间：1-10小时``

用户为lfs，终端输入

```
cd $LFS/sources # 进入到下载的文件夹
for file in *.tar.xz; do tar -xf "$file"; done # 解压所有的tar.xz文件

```

## 4.1. Binutils 第一遍

要在lfs用户下运行并且确保在build目录下。这个build目录是在binutils目录下创建的。

``预计时间：5分钟``

运行

```
mkdir -v build
cd build
../configure --prefix=$LFS/tools \
--with-sysroot=$LFS \
--target=$LFS_TGT \
--disable-nls \
--enable-gprofng=no \
--disable-werror \
--enable-new-dtags \
--enable-default-hash-style=gnu
```

如果出现错误，如下图所示，请检查前置条件是否满足。比如上方的环境检查脚本。

![](https://img.164314.xyz/2024/09/ebef0b5da9c6e4751eae3ba5d458fbb5.png)


接下来运行

```
make && make install # 编译并安装。此前提为已经运行了上方的configure命令。并且当前目录是在build目录下。
```

如出现以下错误，可以运行 `sudo apt install texinfo`来解决。当然，这个错误理应在前面的环境检查中就已经解决了。
![](https://img.164314.xyz/2024/09/125cc1a8e3dc08d08164c1e535f97684.png)

### 安装完毕后，可以看到如下图所示。

![](https://img.164314.xyz/2024/09/d9587277be90d1cc178ea882360a5f99.png)

## 4.2. GCC

``预计时间：30分钟``

gcc依赖于gmp、mpfr、mpc所以要先安装这些依赖。

因为前面我们已经下载了并且解压了这三个依赖，所以我们可以直接进行编译。只需要把这三个依赖放到gcc的目录下即可。
具体方法为使用mv命令，按照mv gmp-版本号 gcc-版本号/gmp来移动gmp到gcc目录下。mpfr和mpc同理。

需要注意的是，这里的版本号是你下载的版本号，移动过去的文件是不带版本号的。只有一个gmp、mpfr、mpc的名字的文件夹。

    如果你是x86_64的系统，需要运行`case $(uname -m) in
    x86_64)
    sed -e '/m64=/s/lib64/lib/' \
    -i.orig gcc/config/i386/t-linux64
    ;;
    esac`

接下来cd到gcc目录下（终端输入cd gcc+tab可以自动补全），然后运行

```
mkdir -v build
cd build

../configure \
--target=$LFS_TGT \
--prefix=$LFS/tools \
--with-glibc-version=2.40 \
--with-sysroot=$LFS \
--with-newlib \
--without-headers \
--enable-default-pie \
--enable-default-ssp \
--disable-nls \
--disable-shared \
--disable-multilib \
--disable-threads \
--disable-libatomic \
--disable-libgomp \
--disable-libquadmath \
--disable-libssp \
--disable-libvtv \
--disable-libstdcxx \
--enable-languages=c,c++
```

小趣事：在这里我的vmware因为内存不足需要提升一下，我重启了一下vmware，然后把内存提升到了4G。再次进入系统的时候进入lfs目录我发现我文件全没了。我突然就慌了，然后检查了一下发现原来是我没有mount分区，然后我就mount了一下，然后文件全回来了。哈哈哈

运行完上面的命令后，运行

```
make && make install
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
`dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h # 在gcc目录下运行
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/a111f846ea0594241ec0961532e05246.png)

或者使用ls $LFS/tools/bin来查看是否有gcc和g++。

![](https://img.164314.xyz/2024/09/5bd1559c3af73c91e7b4d8a21ba0649d.png)

## 4.3 Linux Api header

``预计时间：1分钟``

先cd到linux-6.10.5目录下，然后依次运行

```
make mrproper
make headers
find usr/include -type f ! -name '*.h' -delete
cp -rv usr/include $LFS/usr
```

![](https://img.164314.xyz/2024/09/1c6a8ec2cf11272b807af1c39815eaee.png)

## 4.4 Glibc

``预计时间：15分钟``



先运行一下下方的命令。请注意这里，**非常的重要**。如果前面的前置条件没有满足，下面的命令会造成系统崩溃。

建议：在运行前创建一个快照。在运行，如果出错，可以直接恢复快照。

```
case $(uname -m) in
i?86) ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
;;
x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
;;
esac
```
然后进入到glibc文件夹中，使用cd。

进入后输入命令 `patch -Np1 -i ../glibc-2.40-fhs-1.patch`即可自动完成补丁。请注意这里的补丁是在前面我们就已经下载好的文件，应该在sources文件夹下面。


![](https://img.164314.xyz/2024/09/bc9192ca4ae935d122ef2c12b0c0638f.png)


在运行configure前我们的系统中应该有两个依赖包，分别是gawk bison & gettext。让我们用apt-get install 来安装它们。如果前面已经安装完毕请忽略这一步。

```
apt-get gawk bison
apt-get install gettext
```

接下来运行configure（build目录下）

```
../configure \
--prefix=/usr \
--host=$LFS_TGT \
--build=$(../scripts/config.guess) \
--enable-kernel=4.19 \
--with-headers=$LFS/usr/include \
--disable-nscd \
libc_cv_slibdir=/usr/lib
```

成功截图如下

![](https://img.164314.xyz/2024/09/93e27f6e07ec8dd2ec072d2bb86f07ea.png)

在configure成功后，运行

```
make && make DESTDIR=$LFS install
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
```

到此就安装好了，接下来可以检查一下是否安装成功。

`echo 'int main(){}' | $LFS_TGT-gcc -xc -
readelf -l a.out | grep ld-linux`

结果如下图所示。

![](https://img.164314.xyz/2024/09/6285329fb7d3285793d114ca6e37fd4b.png)

最后删除一下临时文件 `rm -v a.out`

## 4.5. Libstdc++

``预计时间：1分钟``
此文件是在gcc中的，所以再次进入gcc的目录下，创建一个新的文件夹，然后运行configure

运行

```
$LFS/sources/gcc-14.2.0/libstdc++-v3/configure --host=$LFS_TGT --build=$(../config.guess) --prefix=/usr --disable-multilib --disable-nls --disable-libstdcxx-pch --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/14.2.0

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/ff2d7c415f7dedf3c53b1a7931b1075b.png)

## 4.6 M4
    
``预计时间：1分钟``

进入m4目录，然后运行

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make

make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/911ac368e189b5d42c1393a545da8d0a.png)

## 4.7. Ncurses

``预计时间：5分钟``

运行以下几条命令


```
sed -i s/mawk// configure

mkdir build
pushd build
../configure
make -C include
make -C progs tic
popd

./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(./config.guess) \
--mandir=/usr/share/man \
--with-manpage-format=normal \
--with-shared \
--without-normal \
--with-cxx-shared \
--without-debug \
--without-ada \
--disable-stripping

make 

make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
-i $LFS/usr/include/curses.h
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/00420f8096763eaf5b3633fa874ae77f.png)

## 4.8. Bash

``预计时间：5分钟``

运行以下几条命令（所有的configure都是在包的目录下面才能运行）

```
./configure --prefix=/usr \
--build=$(sh support/config.guess) \
--host=$LFS_TGT \
--without-bash-malloc \
bash_cv_strtold_broken=no

make

make DESTDIR=$LFS install

ln -sv bash $LFS/bin/sh
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/649a9eae201ae008ff0a9c17c18698aa.png)

## 4.9. Coreutils

``预计时间：5分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess) \
--enable-install-program=hostname \
--enable-no-install-program=kill,uptime

make

make DESTDIR=$LFS install

mv -v $LFS/usr/bin/chroot $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' $LFS/usr/share/man/man8/chroot.8
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/cc12b957fa3f4eac9bc302b4c776a3d0.png)

## 4.10. Diffutils

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(./build-aux/config.guess)

make

make DESTDIR=$LFS install

```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/7b5325a2c73dca0fefda846067520cb4.png)

## 4.11. File

``预计时间：1分钟``

运行以下几条命令

```
mkdir build

pushd build
../configure --disable-bzlib \
--disable-libseccomp \
--disable-xzlib \
--disable-zlib
make
popd

./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)

make FILE_COMPILE=$(pwd)/build/src/file

make DESTDIR=$LFS install

rm -v $LFS/usr/lib/libmagic.la
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/3d96da6fb7356e89616816134c0a6ad6.png)

## 4.12. Findutils

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--localstatedir=/var/lib/locate \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make

make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/8b03108b0608e2620d9e178d35ded225.png)

## 4.13. Gawk

``预计时间：1分钟``

运行以下几条命令

```
sed -i 's/extras//' Makefile.in

./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/2bbe99910ad061d8540d31dde6f0bc5c.png)

## 4.14. Grep

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(./build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/f1b3318bc65c4c90eb28749c0f388b3c.png)

## 4.15. Gzip

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr --host=$LFS_TGT

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/2ad1a53bdaab0c6674f2424ea2805704.png)

## 4.16. Make

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--without-guile \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/d1f66a726d343c76240af2b3be7691e5.png)

## 4.17. Patch

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/a8e26fa63a84f59b99059fac6d6e064b.png)

## 4.18. Sed

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(./build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/e7a6ad4f94b6c940fdbda8691ad45691.png)

## 4.19. Tar

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess)

make && make DESTDIR=$LFS install
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/85a437940ba55dc7d2fe0695b3e7f88a.png)


## 4.20. Xz

``预计时间：1分钟``

运行以下几条命令

```
./configure --prefix=/usr \
--host=$LFS_TGT \
--build=$(build-aux/config.guess) \
--disable-static \
--docdir=/usr/share/doc/xz-5.6.2

make && make DESTDIR=$LFS install

rm -v $LFS/usr/lib/liblzma.la
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/4250c73aa3e353ffe1a38d54afe8e5f0.png)

## 4.21. Binutils 第二遍

``预计时间：5分钟``

运行以下几条命令

```
sed '6009s/$add_dir//' -i ltmain.sh

mkdir -v new_build
cd new_build

../configure \
--prefix=/usr \
--build=$(../config.guess) \
--host=$LFS_TGT \
--disable-nls \
--enable-shared \
--enable-gprofng=no \
--disable-werror \
--enable-64-bit-bfd \
--enable-new-dtags \
--enable-default-hash-style=gnu

make && make DESTDIR=$LFS install

rm -v $LFS/usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes,sframe}.{a,la}
```

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/3a12ae3d0939378bee887b62ef40b1d0.png)


## 4.22. Gcc 第二遍

``预计时间：30分钟``


运行以下几条命令

```

tar -xf ../mpfr-4.2.1.tar.xz
mv -v mpfr-4.2.1 mpfr
tar -xf ../gmp-6.3.0.tar.xz
mv -v gmp-6.3.0 gmp
tar -xf ../mpc-1.3.1.tar.gz
mv -v mpc-1.3.1 mpc

case $(uname -m) in
x86_64)
sed -e '/m64=/s/lib64/lib/' \
-i.orig gcc/config/i386/t-linux64
;;
esac

sed '/thread_header =/s/@.*@/gthr-posix.h/' \
-i libgcc/Makefile.in libstdc++-v3/include/Makefile.in

mkdir -v new_build
cd new_build

../configure \
--build=$(../config.guess) \
--host=$LFS_TGT \
--target=$LFS_TGT \
LDFLAGS_FOR_TARGET=-L$PWD/$LFS_TGT/libgcc \
--prefix=/usr \
--with-build-sysroot=$LFS \
--enable-default-pie \
--enable-default-ssp \
--disable-nls \
--disable-multilib \
--disable-libatomic \
--disable-libgomp \
--disable-libquadmath \
--disable-libsanitizer \
--disable-libssp \
--disable-libvtv \
--enable-languages=c,c++

make

make DESTDIR=$LFS install

ln -sv gcc $LFS/usr/bin/cc
```


运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/9ac087078e5f32906e95bc69ee59bfe9.png)


# 5. 进⼊ Chroot 并构建其他临时⼯具

在进行下一步之前，我非常建议你创建一个快照。因为下一步如果出现错误，你可以直接恢复快照。相信如果你一步一步的做到这里了，你一定不想因为一个小错误而重头再来。


## 5.1 更改权限

在$LFS目录下的用户组是lfs。在进行后续的操作时，我们需要更改一下权限。在那之前，记得进入root用户。

```
chown --from lfs -R root:root $LFS/{usr,lib,var,etc,bin,sbin,tools}
case $(uname -m) in
x86_64) chown --from lfs -R root:root $LFS/lib64 ;;
esac
```

运行后的截图如下

![](https://img.164314.xyz/2024/09/7633e5c3ac92c02733389b3f30661676.png)

## 5.2 准备虚拟内核⽂件系统

运行以下命令
```
mkdir -pv $LFS/{dev,proc,sys,run}
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

if [ -h $LFS/dev/shm ]; then
install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

![](https://img.164314.xyz/2024/09/434af6b664574ae5ba7a410aa4563b89.png)


![](https://img.164314.xyz/2024/09/91528ac91135908353cf679381b265b8.png)

## 5.3 进入 Chroot 环境

终于到了这一步了，我们要进入chroot环境了。在这个环境下，我们可以在$LFS目录下进行操作，而不会影响到我们的主系统。

```
chroot "$LFS" /usr/bin/env -i \
HOME=/root \
TERM="$TERM" \
PS1='(lfs chroot) \u:\w\$ ' \
PATH=/usr/bin:/usr/sbin \
MAKEFLAGS="-j$(nproc)" \
TESTSUITEFLAGS="-j$(nproc)" \
/bin/bash --login
``` 

运行成功后如下图所示。

![](https://img.164314.xyz/2024/09/051f52a54a68642b8f3cf6189cf7deac.png)

## 5.4 创建chroot的目录

```
mkdir -pv /{boot,home,mnt,opt,srv}
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/lib/locale
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}
ln -sfv /run /var/run
ln -sfv /run/lock /var/lock
install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```

![](https://img.164314.xyz/2024/09/c6e947d8921dbfe8cbc80b8e551aeb39.png)

## 5.5 创建必要的⽂件和符号链接

接下来我们要创建一些必要的文件和符号链接。然后添加一个root用户并且去除I have no name!的提示。
```
ln -sv /proc/self/mounts /etc/mtab
cat > /etc/hosts << EOF
127.0.0.1 localhost $(hostname)
::1 localhost
EOF

cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF

cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF

localedef -i C -f UTF-8 C.UTF-8

echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
install -o tester -d /home/tester

exec /usr/bin/bash --login
```

运行上方的命令后，你会看到如下图所示。

![](https://img.164314.xyz/2024/09/b0df74b8392d8568be4b45ddd2e11861.png)

现在需要来初始化一些文件

```
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664 /var/log/lastlog
chmod -v 600 /var/log/btmp
```

![](https://img.164314.xyz/2024/09/9d55869fab3a219a103bc797656d5132.png)

# 6. 安装基本系统软件

在这里说一句，如果系统重启了的话，再次进入chroot环境的时候，你需要重新mount一下。也就是再次挂载一下。这个刚上手的一天是绝对搞不好的。

---

终于来到这一步了，但是系统中还是有很多的命令是无法使用的，这就相当于一个刚出生的婴儿，什么都不会，你需要去教他怎么去做。当然，在这里就相当于我们给这个婴儿装上了一些基本的软件，让他能够做一些基本的事情。

## 6.1. Gettext

在正式开始安装之前，需要先将你前面所下载的文件包挂载到chroot环境中。比如我这里的文件包都在/mnt/lfs/sources下，所以我需要把这个文件夹挂载到chroot环境中。

```
mount -v --bind /mnt/lfs/sources "$LFS"/mnt
```

这样子在进入后，你就可以在/mnt下找到你的文件包了。直接进是没有任何东西的。直到现在我才知道平时简单的命令如果突然不见了，那是多么的不方便。

接下来开始编译gettext。先确认一下这个文件是否在/mnt下。


运行

```
./configure --disable-shared

make

cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin
```

这里只编译不安装。

成功后如下图所示。

![](https://img.164314.xyz/2024/09/9708be6395b331d005c71612e4b97ea1.png)

## 6.2. Bison

运行
```
./configure --prefix=/usr \
--docdir=/usr/share/doc/bison-3.8.2

make && make install
```

成功后如下图所示。
![](https://img.164314.xyz/2024/09/1a66fccccebe6930ff521ea04297ab7e.png)

## 6.3. Perl

运行
```

sh Configure -des \
-D prefix=/usr \
-D vendorprefix=/usr \
-D useshrplib \
-D privlib=/usr/lib/perl5/5.40/core_perl \
-D archlib=/usr/lib/perl5/5.40/core_perl \
-D sitelib=/usr/lib/perl5/5.40/site_perl \
-D sitearch=/usr/lib/perl5/5.40/site_perl \
-D vendorlib=/usr/lib/perl5/5.40/vendor_perl \
-D vendorarch=/usr/lib/perl5/5.40/vendor_perl

make && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/e1d498416e52780159f1f873d248534e.png)


## 6.4. Python

运行

```
./configure --prefix=/usr \
--enable-shared \
--without-ensurepip

make && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/bbcb8b1dd557c7c7b7e6f38d01e76643.png)

## 6.5. Texinfo

运行
```
./configure --prefix=/usr

make && make install 
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/e9bf57f11efacdc13122af44eb73b1ec.png)


## 6.6. Util-linux

运行
```

mkdir -pv /var/lib/hwclock

./configure --libdir=/usr/lib \
--runstatedir=/run \
--disable-chfn-chsh \
--disable-login \
--disable-nologin \
--disable-su \
--disable-setpriv \
--disable-runuser \
--disable-pylibmount \
--disable-static \
--disable-liblastlog2 \
--without-python \
ADJTIME_PATH=/var/lib/hwclock/adjtime \
--docdir=/usr/share/doc/util-linux-2.40.2

make && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/e9d38d4f26e02b2d07a7c0eced701cb2.png)

## 6.7. 清理

到这里，我们已经安装了基本的环境，接下来我们要清理一下临时文件

运行
```
rm -rf /usr/share/{info,man,doc}/*

find /usr/{lib,libexec} -name \*.la -delete

rm -rf /tools
```

最后一步，退出chroot环境（如果你选择使用命令行进行备份，如果不是，请直接在chroot环境下开始下一章的内容）

```
exit
```

另外不要忘了解除挂载的文件夹

``` 
mountpoint -q $LFS/dev/shm && umount $LFS/dev/shm
umount $LFS/dev/pts
umount $LFS/{sys,proc,run,dev}
```

![](https://img.164314.xyz/2024/09/8f3b648cebbf490591e494aa545c5ca5.png)


**非常建议在这里进行一次快照，如果你使用的是虚拟机。真机请用以下命令进行备份**

```
cd $LFS
tar -cJpf $HOME/lfs-temp-tools-12.2.tar.xz .
```

**恢复命令如下**

```
cd $LFS
rm -rf ./* # 非常危险的命令，请注意LFS环境变量是正常的，不建议直接复制，而是一步一步来输入。非常危险！！
tar -xpf $HOME/lfs-temp-tools-12.2.tar.xz
```


# 7. 构建 LFS 系统

终于到了这里了。如果你是一步一步手动进行操作的，那么你一定会非常的开心，事实上我也是，终于来到这里了，前面的有很多次都忘记了进行备份，导致重头再来。想想都不想再来一次了。哈哈。

再次声明：**我的操作皆是书上的操作。** 建议和书在和此文一起看，会有很多的方便。很多地方只需要直接复制粘贴就可以了。


## 安装基本系统软件

在这里我们要安装基本的系统软件，这里的软件是我们在构建LFS系统时需要的软件。这里的软件是我们在构建LFS系统时需要的软件。这里的软件是我们在构建LFS系统时需要的软件。重要的事情说三遍。

**记得使用root用户来运行。是在chroot下的root，不是真机root！！！**

**记得使用root用户来运行。是在chroot下的root，不是真机root！！！**

**记得使用root用户来运行。是在chroot下的root，不是真机root！！！**

**所有的软件包都在根目录下的sources文件夹中**

**所有的软件包都在根目录下的sources文件夹中**

**所有的软件包都在根目录下的sources文件夹中**


## 7.1. Man-pages

运行
```
rm -v man3/crypt*

make prefix=/usr install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/5f69768afdac42b9ff146723c0db10de.png)

## 7.2 Iana-Etc

这里只需要复制一下文件就可以了。

运行
```
cp services protocols /etc
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/cb14cc5828800aab2bf2750c7bd76ac0.png)

## 7.3. Glibc

进入到根目录下的sources目录中，先来运行一下打补丁的操作

```
patch -Np1 -i ../glibc-2.40-fhs-1.patch
```

如图所示

![](https://img.164314.xyz/2024/09/4e7c7100b53c18f8caef38e31a0a7315.png)

再依次运行进行编译，测试和安装

```
mkdir -v t_build
cd t_build

echo "rootsbindir=/usr/sbin" > configparms

../configure --prefix=/usr \
--disable-werror \
--enable-kernel=4.19 \
--enable-stack-protector=strong \
--disable-nscd \
libc_cv_slibdir=/usr/lib

make

make check # 在测试中可能会出现几个错误，但是无伤大雅，下图是详细信息
```

![](https://img.164314.xyz/2024/09/c4000f90320cf4e5d279281f168288aa.png)

### 7.3.1 install glibc

在进行上方的check后，接下来我们就可以运行make install了，不得不说，check的时间特别的久...

在这里可以看到check的结果。不用去管，我们来运行

`sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile`

和

`make install`。


![](https://img.164314.xyz/2024/09/285c287795c575ce64568e4f853e6f56.png)


![](https://img.164314.xyz/2024/09/c33c4aee59f1d2dd884eb1dc2eebaa3b.png)

### 7.3.2 install locale

虽然这一步可以省略，但是有可能在进行check的时候会出现错误，所以我们还是运行一下。

```
localedef -i C -f UTF-8 C.UTF-8
localedef -i cs_CZ -f UTF-8 cs_CZ.UTF-8
localedef -i de_DE -f ISO-8859-1 de_DE
localedef -i de_DE@euro -f ISO-8859-15 de_DE@euro
localedef -i de_DE -f UTF-8 de_DE.UTF-8
localedef -i el_GR -f ISO-8859-7 el_GR
localedef -i en_GB -f ISO-8859-1 en_GB
localedef -i en_GB -f UTF-8 en_GB.UTF-8
localedef -i en_HK -f ISO-8859-1 en_HK
localedef -i en_PH -f ISO-8859-1 en_PH
localedef -i en_US -f ISO-8859-1 en_US
localedef -i en_US -f UTF-8 en_US.UTF-8
localedef -i es_ES -f ISO-8859-15 es_ES@euro
localedef -i es_MX -f ISO-8859-1 es_MX
localedef -i fa_IR -f UTF-8 fa_IR
localedef -i fr_FR -f ISO-8859-1 fr_FR
localedef -i fr_FR@euro -f ISO-8859-15 fr_FR@euro
localedef -i fr_FR -f UTF-8 fr_FR.UTF-8
localedef -i is_IS -f ISO-8859-1 is_IS
localedef -i is_IS -f UTF-8 is_IS.UTF-8
localedef -i it_IT -f ISO-8859-1 it_IT
localedef -i it_IT -f ISO-8859-15 it_IT@euro
localedef -i it_IT -f UTF-8 it_IT.UTF-8
localedef -i ja_JP -f EUC-JP ja_JP
localedef -i ja_JP -f SHIFT_JIS ja_JP.SJIS 2> /dev/null || true
localedef -i ja_JP -f UTF-8 ja_JP.UTF-8
localedef -i nl_NL@euro -f ISO-8859-15 nl_NL@euro
localedef -i ru_RU -f KOI8-R ru_RU.KOI8-R
localedef -i ru_RU -f UTF-8 ru_RU.UTF-8
localedef -i se_NO -f UTF-8 se_NO.UTF-8
localedef -i ta_IN -f UTF-8 ta_IN.UTF-8
localedef -i tr_TR -f UTF-8 tr_TR.UTF-8
localedef -i zh_CN -f GB18030 zh_CN.GB18030
localedef -i zh_HK -f BIG5-HKSCS zh_HK.BIG5-HKSCS
localedef -i zh_TW -f UTF-8 zh_TW.UTF-8

make localedata/install-locales

localedef -i C -f UTF-8 C.UTF-8
localedef -i ja_JP -f SHIFT_JIS ja_JP.SJIS 2> /dev/null || true
```

![](https://img.164314.xyz/2024/09/d0aedb780cd33ddc12496d70bf9541b1.png)



### 7.3.3 配置glibc

#### 创建/etc/nsswitch.conf

```
cat > /etc/nsswitch.conf << "EOF"
# Begin /etc/nsswitch.conf
passwd: files
group: files
shadow: files
hosts: files dns
networks: files
protocols: files
services: files
ethers: files
rpc: files
# End /etc/nsswitch.conf
EOF
```

#### 添加时区数据

```
tar -xf ../../tzdata2024a.tar.gz
ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right}
for tz in etcetera southamerica northamerica europe africa antarctica \
asia australasia backward; do
zic -L /dev/null -d $ZONEINFO ${tz}
zic -L /dev/null -d $ZONEINFO/posix ${tz}
zic -L leapseconds -d $ZONEINFO/right ${tz}
done
cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
zic -d $ZONEINFO -p America/New_York
unset ZONEINFO
```

![](https://img.164314.xyz/2024/09/652dd1a2c64a11ec4c47a39380f1e549.png)

在运行tzselect后，你会看到如下图所示。

![](https://img.164314.xyz/2024/09/d47c3ec442fa27a6a3d5928e7864b41e.png)

输出了一个时区，你可以根据你的时区来选择。我这里选择的是Europe/Madrid。

在运行`ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime`后，你会看到如下图所示。

![](https://img.164314.xyz/2024/09/861106464caaceb16b57d07eab2e086a.png)

这说明我们已经为我们的lfs系统设置了时区。好耶！！

#### 配置动态加载器

动态加载器是一个用于加载共享库的程序。我们需要配置一下。

```
cat > /etc/ld.so.conf << "EOF"
# Begin /etc/ld.so.conf
/usr/local/lib
/opt/lib
EOF

cat >> /etc/ld.so.conf << "EOF"
# Add an include directory
include /etc/ld.so.conf.d/*.conf
EOF
mkdir -pv /etc/ld.so.conf.d
```

![](https://img.164314.xyz/2024/09/e4f89d81aa942adfa36cdd7b9451bd01.png)

下图是一张安装glibc后的我们能使用的命令。

![](https://img.164314.xyz/2024/09/d9e3db3b4b69bbeb612ba3d48b781c96.png)

**到此为止，我们已经完成了glibc的安装。好耶！！！！**
---

## 7.4. Zlib

用来解压缩文件的。

运行
```
./configure --prefix=/usr

make

make check

make install
```

make check的结果如下图所示。

![](https://img.164314.xyz/2024/09/ef928b762204939a743550041aa6f791.png)

make install的结果如下图所示。

![](https://img.164314.xyz/2024/09/b3739472df1d1c76adb56180f08b435b.png)

删除一下无用的文件。

```
rm -fv /usr/lib/libz.a
```

---
## 7.5. Bzip2

也是用来解压缩文件的。

运行
```
patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch # 打补丁

sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile # 确认安装的符号链接是相对

sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile 

make -f Makefile-libbz2_so
make clean

make # 编译Bzip2

make PREFIX=/usr install # 安装Bzip2

cp -av libbz2.so.* /usr/lib
ln -sv libbz2.so.1.0.8 /usr/lib/libbz2.so # 共享库

cp -v bzip2-shared /usr/bin/bzip2
for i in /usr/bin/{bzcat,bunzip2}; do
ln -sfv bzip2 $i
done

rm -fv /usr/lib/libbz2.a # 删除静态库
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/1757aae133b0365cda2603d001084aa6.png)

下图是Bzip2的命令。

![](https://img.164314.xyz/2024/09/7e68622a2a11e6f67e60e6de97ee350d.png)

## 7.6. Xz

还是用来解压缩文件的。

运行
```
./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/xz-5.6.2

make && make check && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/b7e1f25039babac76b2a15f6d6508ce9.png)

命令如下

![](https://img.164314.xyz/2024/09/e7597ad6dd126842cd441e2e76ede44b.png)

## 7.7 lz4

一种压缩算法。

运行
```
make BUILD_STATIC=no PREFIX=/usr

make -j1 check

make BUILD_STATIC=no PREFIX=/usr install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/e6f99e1af8a7a28dd5651ab026ea41e5.png)

![](https://img.164314.xyz/2024/09/68108bad0de59c0f81b654dc5e55fff5.png)

## 7.8. Zstd

一种实时压缩算法

运行
```
make prefix=/usr

make check # 可能会出现failed的情况，不用管。又不是fail

make prefix=/usr install # 安装

rm -v /usr/lib/libzstd.a # 删除静态库
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/d1fbfe731b72759bfa87b9566b0016fa.png)

![](https://img.164314.xyz/2024/09/1a7e09e054fbac7af08a0695d42a29cb.png)

## 7.9. File

用来检查文件类型的。

运行
```
./configure --prefix=/usr

make

make check

make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/b62005a08ad302c83bfefc8235037d4b.png)

## 7.10. Readline

运行
```
sed -i '/MV.*old/d' Makefile.in
sed -i '/{OLDSUFF}/c:' support/shlib-install

sed -i 's/-Wl,-rpath,[^ ]*//' support/shobj-conf

./configure --prefix=/usr \
--disable-static \
--with-curses \
--docdir=/usr/share/doc/readline-8.2.13

make SHLIB_LIBS="-lncursesw"

make SHLIB_LIBS="-lncursesw" install

install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.2.13

```

![](https://img.164314.xyz/2024/09/79849098a83a51a32cead59bf44d11f7.png)

## 7.11. M4

运行
```
./configure --prefix=/usr

make && make check && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/13277f859793ca329ada91160e278821.png)

## 7.12. Bc

运行
```
CC=gcc ./configure --prefix=/usr -G -O3 -r

make && make test && make install
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/b0e392c0d8d8b88794c62e9f26e6d884.png)

## 7.13. Flex

运行
```
./configure --prefix=/usr \
--docdir=/usr/share/doc/flex-2.6.4 \
--disable-static

make && make check && make install

ln -sv flex /usr/bin/lex
ln -sv flex.1 /usr/share/man/man1/lex.1
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/a813ee06d40a321b6435cf99ae5c382b.png)

![](https://img.164314.xyz/2024/09/5513f4707432d2a9689381d85950795e.png)


## 7.14. tcl

Tcl 软件包包含⼯具命令语⾔，它是⼀个可靠的通⽤脚本语⾔

运行
```
SRCDIR=$(pwd)
cd unix
./configure --prefix=/usr \
--mandir=/usr/share/man \
--disable-rpath

make
sed -e "s|$SRCDIR/unix|/usr/lib|" \
-e "s|$SRCDIR|/usr/include|" \
-i tclConfig.sh
sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.7|/usr/lib/tdbc1.1.7|" \
-e "s|$SRCDIR/pkgs/tdbc1.1.7/generic|/usr/include|" \
-e "s|$SRCDIR/pkgs/tdbc1.1.7/library|/usr/lib/tcl8.6|" \
-e "s|$SRCDIR/pkgs/tdbc1.1.7|/usr/include|" \
-i pkgs/tdbc1.1.7/tdbcConfig.sh
sed -e "s|$SRCDIR/unix/pkgs/itcl4.2.4|/usr/lib/itcl4.2.4|" \
-e "s|$SRCDIR/pkgs/itcl4.2.4/generic|/usr/include|" \
-e "s|$SRCDIR/pkgs/itcl4.2.4|/usr/include|" \
-i pkgs/itcl4.2.4/itclConfig.sh
unset SRCDIR

make test

make install

chmod -v u+w /usr/lib/libtcl8.6.so

make install-private-headers

ln -sfv tclsh8.6 /usr/bin/tclsh

mv /usr/share/man/man3/{Thread,Tcl_Thread}.3

cd ..
tar -xf ../tcl8.6.14-html.tar.gz --strip-components=1
mkdir -v -p /usr/share/doc/tcl-8.6.14
cp -v -r ./html/* /usr/share/doc/tcl-8.6.14

```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/1f944f96528d0cb62b1447bdf3a9f1a8.png)

![](https://img.164314.xyz/2024/09/b08889b6ecde0d2b68e61ae8936b8581.png)

## 7.15. Expect

Expect 软件包包含通过脚本控制的对话，⾃动化 telnet，ftp，passwd，fsck，rlogin，以及
tip 等交互应⽤的⼯具。Expect 对于测试这类程序也很有⽤，它简化了这类通过其他⽅式很难完成的⼯
作。DejaGnu 框架是使⽤ Expect 编写的

运行
```
python3 -c 'from pty import spawn; spawn(["echo", "ok"])' # 检测环境是否正常
```

应该输出OK，如图所示。如果输出不是OK，那么请检查是否已经正确了挂载devpts文件系统。

![](https://img.164314.xyz/2024/09/2e59a1c1568e8e6d1dffefab329cde78.png)

接下来运行

```
patch -Np1 -i ../expect-5.45.4-gcc14-1.patch

./configure --prefix=/usr \
--with-tcl=/usr/lib \
--enable-shared \
--disable-rpath \
--mandir=/usr/share/man \
--with-tclinclude=/usr/include

make

make test

make install
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib

```

![](https://img.164314.xyz/2024/09/8968813cfefbd2724395d466e8d10552.png)

## 7.16. DejaGNU

运行
```
mkdir -v build
cd build

../configure --prefix=/usr
makeinfo --html --no-split -o doc/dejagnu.html ../doc/dejagnu.texi
makeinfo --plaintext -o doc/dejagnu.txt ../doc/dejagnu.texi

make check

make install
install -v -dm755 /usr/share/doc/dejagnu-1.6.3
install -v -m644 doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.3

```

![](https://img.164314.xyz/2024/09/6f61f62aaf0f4074670f8242c9dd94c6.png)

## 7.17 pkg-config

运行
```
./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/pkgconf-2.3.0

make  && make install
```

![](https://img.164314.xyz/2024/09/a9536b7b21b099819a3fb1a1840d9a84.png)

## 7.18 Binutils

运行
```

mkdir -v build
cd build

../configure --prefix=/usr \
--sysconfdir=/etc \
--enable-gold \
--enable-ld=default \
--enable-plugins \
--enable-shared \
--disable-werror \
--enable-64-bit-bfd \
--enable-new-dtags \
--with-system-zlib \
--enable-default-hash-style=gnu

make tooldir=/usr

make -k check

make tooldir=/usr install
    
```
![](https://img.164314.xyz/2024/09/76d576de35a87fc0acbf3cae02ed94fa.png)

![](https://img.164314.xyz/2024/09/689c782f310d45bbcae802e07526c2af.png)


## 7.19. GMP

运行
```
./configure --prefix=/usr \
--enable-cxx \
--disable-static \
--docdir=/usr/share/doc/gmp-6.3.0

make
make html

make check 2>&1 | tee gmp-check-log

awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log # 至少要有199个通过

make install
make install-html
```
![](https://img.164314.xyz/2024/09/436e6ae9352a40d86c90cad8976d7a14.png)

![](https://img.164314.xyz/2024/09/22ee394568c6ee9130f8febce4593b15.png)


## 7.20. MPFR

运行
```
./configure --prefix=/usr \
--disable-static \
--enable-thread-safe \
--docdir=/usr/share/doc/mpfr-4.2.1

make
make html

make check

make install
make install-html
```

![](https://img.164314.xyz/2024/09/5726af9beee583f2dce62bdf0177ef23.png)

![](https://img.164314.xyz/2024/09/687ca50ab4db5e3175e2900a483b94be.png)

## 7.21. MPC

运行
```

./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/mpc-1.3.1

make
make html

make check

make install
make install-html
```
![](https://img.164314.xyz/2024/09/f595b688b4a696dc2c85c007e4888b1f.png)

![](https://img.164314.xyz/2024/09/d6bbce927d047f4c9102bfa85dcad7fa.png)

## 7.22. Attr

运行
```
./configure --prefix=/usr \
--disable-static \
--sysconfdir=/etc \
--docdir=/usr/share/doc/attr-2.5.2

make

make check

make install
```

![](https://img.164314.xyz/2024/09/90b1dd3e2107ef341913a7af56ae5b57.png)

![](https://img.164314.xyz/2024/09/ceb1598d8bf11604fef3d1af1b014ef5.png)

## 7.23. Acl

运行
```
./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/acl-2.3.2

make

make install
```

![](https://img.164314.xyz/2024/09/490e816ada4704ee2da6d73766a6df16.png)

![](https://img.164314.xyz/2024/09/487203530dfeedc69cef2c167fe56a5f.png)

## 7.24. Libcap

运行
```
sed -i '/install -m.*STA/d' libcap/Makefile

make prefix=/usr lib=lib

make test

make prefix=/usr lib=lib install
```

![](https://img.164314.xyz/2024/09/d53cbb8ef4a7671d469c47f8c393a37b.png)

![](https://img.164314.xyz/2024/09/675691709c395435f2581a9161828a0c.png)


## 7.25 Libxcrypt

运行
```
./configure --prefix=/usr \
--enable-hashes=strong,glibc \
--enable-obsolete-api=no \
--disable-static \
--disable-failure-tokens

make

make check

make install

make distclean
./configure --prefix=/usr \
--enable-hashes=strong,glibc \
--enable-obsolete-api=glibc \
--disable-static \
--disable-failure-tokens
make
cp -av --remove-destination .libs/libcrypt.so.1* /usr/lib
```

![](https://img.164314.xyz/2024/09/fa7f74d37d28ecec076f21169f8e7b24.png)

![](https://img.164314.xyz/2024/09/b8082d5b4a1b06da9ae3fc73796da0ea.png)


## 7.26 Shadow

运行
```
sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /' {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /' {} \;

sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD YESCRYPT:' \
-e 's:/var/spool/mail:/var/mail:' \
-e '/PATH=/{s@/sbin:@@;s@/bin:@@}' \
-i etc/login.defs

touch /usr/bin/passwd
./configure --sysconfdir=/etc \
--disable-static \
--with-{b,yes}crypt \
--without-libbsd \
--with-group-name-max-length=32

make

make exec_prefix=/usr install
make -C man install-man
```

![](https://img.164314.xyz/2024/09/ce56720967bd015a5766b7415739040e.png)



### 7.26.1 配置Shadow

#### 启用加密

```
pwconv # 对⽤⼾密码启用加密
grpconv # 对组密码启⽤加密

mkdir -p /etc/default
useradd -D --gid 999
```

然后输入`passwd root`来设置root密码。如图所示

![](https://img.164314.xyz/2024/09/83b4a5ac4b166d2c43a82b89e6e8c0d0.png)

## 7.29 Gcc

我在这里系统重启了一下，所以需要重新挂载一下。如果你没有重启，那么你可以直接跳过这一步。

```
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

if [ -h $LFS/dev/shm ]; then
install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi # 

chroot "$LFS" /usr/bin/env -i \
HOME=/root \
TERM="$TERM" \
PS1='(lfs chroot) \u:\w\$ ' \
PATH=/usr/bin:/usr/sbin \
MAKEFLAGS="-j$(nproc)" \
TESTSUITEFLAGS="-j$(nproc)" \
/bin/bash --login # 进入chroot环境
```

运行

```
case $(uname -m) in
x86_64)
sed -e '/m64=/s/lib64/lib/' \
-i.orig gcc/config/i386/t-linux64
;;
esac

mkdir -v t_build
cd t_build

../configure --prefix=/usr \
LD=ld \
--enable-languages=c,c++ \
--enable-default-pie \
--enable-default-ssp \
--enable-host-pie \
--disable-multilib \
--disable-bootstrap \
--disable-fixincludes \
--with-system-zlib

make # 这里可以添加一个-j参数，加快编译速度，比如make -j4

ulimit -s -H unlimited

sed -e '/cpython/d' -i ../gcc/testsuite/gcc.dg/plugin/plugin.exp
sed -e 's/no-pic /&-no-pie /' -i ../gcc/testsuite/gcc.target/i386/pr113689-1.c
sed -e 's/300000/(1|300000)/' -i ../libgomp/testsuite/libgomp.c-c++-common/pr109062.c
sed -e 's/{ target nonpic } //' \
-e '/GOTPCREL/d' -i ../gcc/testsuite/gcc.target/i386/fentryname3.c

chown -R tester .
su tester -c "PATH=$PATH make -k check"

../contrib/test_summary

make install # 这里可以加上time，来查看编译的时间，比如time make install.
```

install的结果如下图所示。

![](https://img.164314.xyz/2024/09/72a1e31725b6fb416bb7c55ec4da59b2.png)

接下来改变一下目录的所有权为root

使用ls -ld命令可以看到所有权目前为tester，我们需要改变为root。

```
ls -ld /usr/lib/gcc/$(gcc -dumpmachine)/14.2.0/include{,-fixed} # 查看所有权

chown -v -R root:root \
/usr/lib/gcc/$(gcc -dumpmachine)/14.2.0/include{,-fixed} # 改变所有权
```

![](https://img.164314.xyz/2024/09/a58f59bd78d75bb5340661f55ee08431.png)

在接下来需要链接一下文件

运行

```
ln -svr /usr/bin/cpp /usr/lib

ln -sv gcc.1 /usr/share/man/man1/cc.1

ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/14.2.0/liblto_plugin.so \
/usr/lib/bfd-plugins/
```

---

如果一切正常，那么在运行测试的时候你会看见如下图所示。

```
echo 'int main(){}' > dummy.c
cc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'

grep -E -o '/usr/lib.*/S?crt[1in].*succeeded' dummy.log 

grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'

grep -B4 '^ /usr/include' dummy.log

grep found dummy.log

grep "/lib.*/libc.so.6 " dummy.log
```

![](https://img.164314.xyz/2024/09/214ded14bd212c9cc4008e7998e8cd95.png)

![](https://img.164314.xyz/2024/09/f6980e62c31387a97579b5ee2c70fea3.png)

![](https://img.164314.xyz/2024/09/2ca61f56d9dea405cc5378dc10093a5e.png)

![](https://img.164314.xyz/2024/09/94b7cbcd8e95d01a7a274ee06f30fbd6.png)

一切正常，删除一些无用的文件。

```
rm -v dummy.c a.out dummy.log
```

移动一个文件到正确的位置。

```
mkdir -pv /usr/share/gdb/auto-load/usr/lib
mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib
```

**至此，我们已经完成了gcc的安装。好耶！！
**
---


## 7.3 Ncurses

运行
```
./configure --prefix=/usr \
--mandir=/usr/share/man \
--with-shared \
--without-debug \
--without-normal \
--with-cxx-shared \
--enable-pc-files \
--with-pkg-config-libdir=/usr/lib/pkgconfig

make

make DESTDIR=$PWD/dest install
install -vm755 dest/usr/lib/libncursesw.so.6.5 /usr/lib
rm -v dest/usr/lib/libncursesw.so.6.5
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
-i dest/usr/include/curses.h
cp -av dest/* /

for lib in ncurses form panel menu ; do
ln -sfv lib${lib}w.so /usr/lib/lib${lib}.so
ln -sfv ${lib}w.pc /usr/lib/pkgconfig/${lib}.pc
done

ln -sfv libncursesw.so /usr/lib/libcurses.so
```

成功后如下图所示。

![](https://img.164314.xyz/2024/09/384a50165b2caffaa78326f8b5ab06c0.png)

![](https://img.164314.xyz/2024/09/649cfb3211f30e1af42d69cf7700c040.png)

## 7.31 Sed

运行

```
./configure --prefix=/usr

make && make html

chown -R tester .
su tester -c "PATH=$PATH make check"

make install
install -d -m755 /usr/share/doc/sed-4.9
install -m644 doc/sed.html /usr/share/doc/sed-4.9
```
![](https://img.164314.xyz/2024/09/300e62ee3d606a209369b49d9e8f8825.png)


## 7.32 Psmisc

运行

```
./configure --prefix=/usr

make && make install
```

![](https://img.164314.xyz/2024/09/00e142b607d3df19030e2fd466cd32c6.png)


## 7.33 Gettext

运行

```
./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/gettext-0.22.5

make

make install
chmod -v 0755 /usr/lib/preloadable_libintl.so
```

![](https://img.164314.xyz/2024/09/aa30455afa30539121d7325f38035183.png)


## 7.34 Bison

运行

```
./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.8.2

make

make check # 如果你想要进行测试的话

make install
```

![](https://img.164314.xyz/2024/09/8146e879ae8a7073d368d1477ca5f03b.png)

## 7.35 Grep

运行

```
sed -i "s/echo/#echo/" src/egrep.sh

./configure --prefix=/usr

make && make install
```

![](https://img.164314.xyz/2024/09/76fad2e5e0f455955c5126c9c92846e6.png)


## 7.36 Bash

运行

```

./configure --prefix=/usr \
--without-bash-malloc \
--with-installed-readline \
bash_cv_strtold_broken=no \
--docdir=/usr/share/doc/bash-5.2.32

make && make install

exec /usr/bin/bash --login
``` 

![](https://img.164314.xyz/2024/09/c507932630f2f23ae801f35b8217af42.png)

## 7.37 Libtool

运行

```
./configure --prefix=/usr

make && make install

rm -fv /usr/lib/libltdl.a
```

![](https://img.164314.xyz/2024/09/bc17b8b53bd11fd4a8cc60962e7a1028.png)

## 7.38 GDBM

运行

```
./configure --prefix=/usr \
--disable-static \
--enable-libgdbm-compat

make && make install
```

![](https://img.164314.xyz/2024/09/5d2c022f9634bfb7e2fcc7cd00ca38e0.png)

## 7.39 Gperf

运行

```
./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.1

make && make install
```

![](https://img.164314.xyz/2024/09/254e50dd6120f4d2648c4b2f5ce59a03.png)

## 7.40 Expat

```
./configure --prefix=/usr \
--disable-static \
--docdir=/usr/share/doc/expat-2.6.2

make && make install
```

![](https://img.164314.xyz/2024/09/1dce04d7ea0bf31eeb7cf5c33a328eaa.png)

## 7.41 Inetutils

```
sed -i 's/def HAVE_TERMCAP_TGETENT/ 1/' telnet/telnet.c

./configure --prefix=/usr \
--bindir=/usr/bin \
--localstatedir=/var \
--disable-logger \
--disable-whois \
--disable-rcp \
--disable-rexec \
--disable-rlogin \
--disable-rsh \
--disable-servers

make 
make install

mv -v /usr/{,s}bin/ifconfig
```

![](https://img.164314.xyz/2024/09/cd1219a16d7a99836fa4b2baab65ac33.png)

## 7.42 less

```
./configure --prefix=/usr --sysconfdir=/etc

make && make install
```

![](https://img.164314.xyz/2024/09/362185502f1fa4e5825a8c8e84609c27.png)

## 7.43 Perl

```
export BUILD_ZLIB=False
export BUILD_BZIP2=0

sh Configure -des \
-D prefix=/usr \
-D vendorprefix=/usr \
-D privlib=/usr/lib/perl5/5.40/core_perl \
-D archlib=/usr/lib/perl5/5.40/core_perl \
-D sitelib=/usr/lib/perl5/5.40/site_perl \
-D sitearch=/usr/lib/perl5/5.40/site_perl \
-D vendorlib=/usr/lib/perl5/5.40/vendor_perl \
-D vendorarch=/usr/lib/perl5/5.40/vendor_perl \
-D man1dir=/usr/share/man/man1 \
-D man3dir=/usr/share/man/man3 \
-D pager="/usr/bin/less -isR" \
-D useshrplib \
-D usethreads

make 
make install
unset BUILD_ZLIB BUILD_BZIP2
```

![](https://img.164314.xyz/2024/09/633eb68707a54fa87b134f10c55fbf3f.png)

![](https://img.164314.xyz/2024/09/04d2f137c4a55baf907e6ab8becbd735.png)

## 7.44 XML::Parser

```
perl Makefile.PL

make && make install
```

![](https://img.164314.xyz/2024/09/b0258fb07c6d55c8f37108894388f495.png)

## 7.45 Intltool

```
sed -i 's:\\\${:\\\$\\{:' intltool-update.in

./configure --prefix=/usr

make 
make install
install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO
```

![](https://img.164314.xyz/2024/09/3121d35e31ec277b24975196a5dfacd1.png)

## 7.46 Autoconf

```
./configure --prefix=/usr

make && make install
```

![](https://img.164314.xyz/2024/09/d17b60108c0d0961027c29db5eeed250.png)


## 7.47 Automake

```
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.17

make
make install
```

![](https://img.164314.xyz/2024/09/cf468f9a74cb774713508fb331374076.png)

## 7.48 openssl

```
./config --prefix=/usr \
--openssldir=/etc/ssl \
--libdir=lib \
shared \
zlib-dynamic

make

sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile
make MANSUFFIX=ssl install

mv -v /usr/share/doc/openssl /usr/share/doc/openssl-3.3.1
cp -vfr doc/* /usr/share/doc/openssl-3.3.1
```

![](https://img.164314.xyz/2024/09/bb87650fde7b2cd2ac8d6d402f4943c9.png)

## 7.49 Kmod
    
```
./configure --prefix=/usr \
--sysconfdir=/etc \
--with-openssl \
--with-xz \
--with-zstd \
--with-zlib \
--disable-manpages

make
make install

for target in depmod insmod modinfo modprobe rmmod; do
ln -sfv ../bin/kmod /usr/sbin/$target
rm -fv /usr/bin/$target
done
``` 

![](https://img.164314.xyz/2024/09/816f7862d621ec674010dd0f3cf8ab5c.png)


## 7.50 Elfutils

```
./configure --prefix=/usr \
--disable-debuginfod \
--enable-libdebuginfod=dummy

make

make -C libelf install
install -vm644 config/libelf.pc /usr/lib/pkgconfig
rm /usr/lib/libelf.a
```

![](https://img.164314.xyz/2024/09/c3c612488b528e2b233114f004476542.png)

## 7.51 Libffi

```
./configure --prefix=/usr \
--disable-static \
--with-gcc-arch=native

make 
make install
```

![](https://img.164314.xyz/2024/09/3932fda170b214af3705a8813343999d.png)


## 7.52 python

```
./configure --prefix=/usr \
--enable-shared \
--with-system-expat \
--enable-optimizations

make
make install
```

![](https://img.164314.xyz/2024/09/fd67627434b0eae703f0474a6c551c30.png)

如果想要禁止一些pip3的信息，可以使用来进行

```
cat > /etc/pip.conf << EOF
[global]
root-user-action = ignore
disable-pip-version-check = true
EOF
```

## 7.53 Flit-core

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --no-user --find-links dist flit_core
```

![](https://img.164314.xyz/2024/09/78877fc6928d416b8b2f1ba40f814de1.png)

## 7.54 wheel

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links=dist wheel
```

![](https://img.164314.xyz/2024/09/14c3b67bf3cbc7d995a528785bbac9d6.png)


## 7.55 setuptools

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist setuptools
```

![](https://img.164314.xyz/2024/09/9025c2815b57d47e4bedcc589f635c90.png)

## 7.56 Ninja

```
python3 configure.py --bootstrap

install -vm755 ninja /usr/bin/
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja
install -vDm644 misc/zsh-completion /usr/share/zsh/site-functions/_ninja
```

![](https://img.164314.xyz/2024/09/9d1f2f7386cf466f9eeabb0e14046e99.png)

## 7.57 Meson

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --find-links dist meson
install -vDm644 data/shell-completions/bash/meson /usr/share/bash-completion/completions/meson
install -vDm644 data/shell-completions/zsh/_meson /usr/share/zsh/site-functions/_meson
```

![](https://img.164314.xyz/2024/09/bca5393892337366cca5096abf596dc6.png)

## 7.58 Coreutils

Coreutils 在make的时候要求aclocal的版本为1.16，但是前面在安装automake的时候安装的是1.17。这里应该就是书中的错误了。所以我们需要重新安装automake，这里是[1.16](https://ftp.gnu.org/gnu/automake/automake-1.16.tar.gz)的网址，先退出chroot环境在进入root环境下的LFS目录下的sources中用wget下载，在进入chroot就能看见了。

重新安装automake1.16

```
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.16
make
make install
```

下面是安装coreutils的命令

patch -Np1 -i ../coreutils-9.5-i18n-2.patch # 这个补丁好像是有一个bug，我这里尝试构建了好几次但是都没有成功，后来我想着不打这个补丁试试，结果成功了。

```
autoreconf -fiv
FORCE_UNSAFE_CONFIGURE=1 ./configure \
--prefix=/usr \
--enable-no-install-program=kill,uptime

make

make install

mv -v /usr/bin/chroot /usr/sbin
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8
```

![](https://img.164314.xyz/2024/09/e0550abb48a10b013cd15c3dca236266.png)


## 7.59 Check

```
./configure --prefix=/usr --disable-static

make

make docdir=/usr/share/doc/check-0.15.2 install
```

![](https://img.164314.xyz/2024/09/c31f9f7fe309cdd165494932930db241.png)


## 7.60 Diffutils

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/b1ad940a92d57b35dd0da6c9c170db02.png)

## 7.61 Gawk

```
sed -i 's/extras//' Makefile.in
./configure --prefix=/usr

make
rm -f /usr/bin/gawk-5.3.0
make install
```

![](https://img.164314.xyz/2024/09/3d9f91578eb511b0ab3fa5e8af51646c.png)

## 7.62 Findutils

```
./configure --prefix=/usr --localstatedir=/var/lib/locate

make

make install
```

![](https://img.164314.xyz/2024/09/3ba9e3d46bd38c970095cc058740e605.png)

## 7.63 Groff

```

PAGE=A4 ./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/8f2a7861c7c9b1652ad1581f2c0803b5.png)

## 7.64 GRUB

```
unset {C,CPP,CXX,LD}FLAGS

echo depends bli part_gpt > grub-core/extra_deps.lst

./configure --prefix=/usr \
--sysconfdir=/etc \
--disable-efiemu \
--disable-werror

make

make install
mv -v /etc/bash_completion.d/grub /usr/share/bash-completion/completions
```

![](https://img.164314.xyz/2024/09/3893bf8cdfaf61981831df20cb32d393.png)

## 7.65 gzip

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/27a61f1e94592ce81c958e34d459012d.png)

## 7.66 IPRoute2

```
sed -i /ARPD/d Makefile
rm -fv man/man8/arpd.8

make NETNS_RUN_DIR=/run/netns

make SBINDIR=/usr/sbin install
```

![](https://img.164314.xyz/2024/09/05326d04f27b9d20524937c4113ec162.png)

## 7.67 Kbd

```
patch -Np1 -i ../kbd-2.6.4-backspace-1.patch

sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in

./configure --prefix=/usr --disable-vlock

make

make install
```

![](https://img.164314.xyz/2024/09/ad03908f7bd9a37a5e91d173e879edb6.png)


## 7.68 libpipeline

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/96fff5e02d85179638a521769dfde9b5.png)

## 7.69 Make

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/56fb988d79e0a97ad666d65c50735c5d.png)

## 7.70 patch

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/4f0a3c3078078944396bad185ec5ee02.png)


## 7.71 tar

```
FORCE_UNSAFE_CONFIGURE=1 \
./configure --prefix=/usr

make

make install
make -C doc install-html docdir=/usr/share/doc/tar-1.35
```

![](https://img.164314.xyz/2024/09/5e5a5a191a93bf913607a42b493e8d14.png)

## 7.72 Texinfo

```
./configure --prefix=/usr

make

make install
```

![](https://img.164314.xyz/2024/09/fd586663330bb018bddb86b583ae5ec8.png)

## 7.73 vim

```
echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h

./configure --prefix=/usr

make

make install

ln -sv vim /usr/bin/vi
for L in /usr/share/man/{,*/}man1/vim.1; do
ln -sv vim.1 $(dirname $L)/vi.1
done # 这里是为了让vi也能使用vim的man手册

ln -sv ../vim/vim91/doc /usr/share/doc/vim-9.1.0660
```

![](https://img.164314.xyz/2024/09/8a7515c35da8093c24adc11be3b0eef3.png)

## 7.731 配置vim
    
```
cat > /etc/vimrc << "EOF"
" Begin /etc/vimrc
" Ensure defaults are set before customizing settings, not after
source $VIMRUNTIME/defaults.vim
let skip_defaults_vim=1
set nocompatible
set backspace=2
set mouse=
syntax on
if (&term == "xterm") || (&term == "putty")
set background=dark
endif
" End /etc/vimrc
EOF
```

## 7.74 MarkupSafe

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --no-user --find-links dist Markupsafe
```

![](https://img.164314.xyz/2024/09/4bc4d65708b066b662c0584f012a841b.png)

## 7.75 Jinja2

```
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD

pip3 install --no-index --no-user --find-links dist Jinja2
```

![](https://img.164314.xyz/2024/09/cd074a0b3e0447f80e547f2fa91447c7.png)

## 7.76 Systemd
    
```
sed -i -e 's/GROUP="render"/GROUP="video"/' \
-e 's/GROUP="sgx", //' rules.d/50-udev-default.rules.in

sed '/systemd-sysctl/s/^/#/' -i rules.d/99-systemd.rules.in

sed '/NETWORK_DIRS/s/systemd/udev/' -i src/basic/path-lookup.h

mkdir -p build
cd build
meson setup .. \
--prefix=/usr \
--buildtype=release \
-D mode=release \
-D dev-kvm-mode=0660 \
-D link-udev-shared=false \
-D logind=false \
-D vconsole=false # 在安装这个时候会有个报错，这时候要先安装Util-linux(7.79)再返回安装这个

ninja udevadm systemd-hwdb \
$(ninja -n | grep -Eo '(src/(lib)?udev|rules.d|hwdb.d)/[^ ]*') \
$(realpath libudev.so --relative-to .) \
$udev_helpers

install -vm755 -d {/usr/lib,/etc}/udev/{hwdb.d,rules.d,network}
install -vm755 -d /usr/{lib,share}/pkgconfig
install -vm755 udevadm /usr/bin/
install -vm755 systemd-hwdb /usr/bin/udev-hwdb
ln -svfn ../bin/udevadm /usr/sbin/udevd
cp -av libudev.so{,*[0-9]} /usr/lib/
install -vm644 ../src/libudev/libudev.h /usr/include/
install -vm644 src/libudev/*.pc /usr/lib/pkgconfig/
install -vm644 src/udev/*.pc /usr/share/pkgconfig/
install -vm644 ../src/udev/udev.conf /etc/udev/
install -vm644 rules.d/* ../rules.d/README /usr/lib/udev/rules.d/
install -vm644 $(find ../rules.d/*.rules \
-not -name '*power-switch*') /usr/lib/udev/rules.d/
install -vm644 hwdb.d/* ../hwdb.d/{*.hwdb,README} /usr/lib/udev/hwdb.d/
install -vm755 $udev_helpers /usr/lib/udev
install -vm644 ../network/99-default.link /usr/lib/udev/network

tar -xvf ../../udev-lfs-20230818.tar.xz
make -f udev-lfs-20230818/Makefile.lfs install

tar -xf ../../systemd-man-pages-256.4.tar.xz \
--no-same-owner --strip-components=1 \
-C /usr/share/man --wildcards '*/udev*' '*/libudev*' \
'*/systemd.link.5' \
'*/systemd-'{hwdb,udevd.service}.8
sed 's|systemd/network|udev/network|' \
/usr/share/man/man5/systemd.link.5 \
> /usr/share/man/man5/udev.link.5
sed 's/systemd\(\\\?-\)/udev\1/' /usr/share/man/man8/systemd-hwdb.8 \
> /usr/share/man/man8/udev-hwdb.8
sed 's|lib.*udevd|sbin/udevd|' \
/usr/share/man/man8/systemd-udevd.service.8 \
> /usr/share/man/man8/udevd.8
rm /usr/share/man/man*/systemd*

unset udev_helpers
```

![](https://img.164314.xyz/2024/09/87232c9518e68ad09440ec814a032166.png)

# 7.77 Man-DB

```
./configure --prefix=/usr \
--docdir=/usr/share/doc/man-db-2.12.1 \
--sysconfdir=/etc \
--disable-setuid \
--enable-cache-owner=bin \
--with-browser=/usr/bin/lynx \
--with-vgrind=/usr/bin/vgrind \
--with-grap=/usr/bin/grap \
--with-systemdtmpfilesdir= \
--with-systemdsystemunitdir=

make 

make install
```

![](https://img.164314.xyz/2024/09/add74d85764ff6148f670cb743e1918a.png)

## 7.78 Procps-ng

```
./configure --prefix=/usr \
--docdir=/usr/share/doc/procps-ng-4.0.4 \
--disable-static \
--disable-kill

make

make install
```

![](https://img.164314.xyz/2024/09/ad3fc5bd174d69c98ad16cc5ca89825e.png)

## 7.79 Util-linux

```
./configure --bindir=/usr/bin \
--libdir=/usr/lib \
--runstatedir=/run \
--sbindir=/usr/sbin \
--disable-chfn-chsh \
--disable-login \
--disable-nologin \
--disable-su \
--disable-setpriv \
--disable-runuser \
--disable-pylibmount \
--disable-liblastlog2 \
--disable-static \
--without-python \
--without-systemd \
--without-systemdsystemunitdir \
ADJTIME_PATH=/var/lib/hwclock/adjtime \
--docdir=/usr/share/doc/util-linux-2.40.2

make 
make install
```

![](https://img.164314.xyz/2024/09/3418f8e3235ce5ec612a187472a0dadb.png)

## 7.80 E2fsprogs

```
mkdir -v build
cd build    

../configure --prefix=/usr \
--sysconfdir=/etc \
--enable-elf-shlibs \
--disable-libblkid \
--disable-libuuid \
--disable-uuidd \
--disable-fsck

make

make install

rm -fv /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a

gunzip -v /usr/share/info/libext2fs.info.gz
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info

```

![](https://img.164314.xyz/2024/09/881f3ef72a9c5323dc481f5a6c04f880.png)

## 7.81 Sysklogd

```
./configure --prefix=/usr \
--sysconfdir=/etc \
--runstatedir=/run \
--without-logger

make

make install
```

```
cat > /etc/syslog.conf << "EOF"
# Begin /etc/syslog.conf
auth,authpriv.* -/var/log/auth.log
*.*;auth,authpriv.none -/var/log/sys.log
daemon.* -/var/log/daemon.log
kern.* -/var/log/kern.log
mail.* -/var/log/mail.log
user.* -/var/log/user.log
*.emerg *
# Do not open any internet ports.
secure_mode 2
# End /etc/syslog.conf
EOF
```

![](https://img.164314.xyz/2024/09/5c7ac84403840bd3ab344729b454e3f8.png)

## 7.82 Sysvinit

```
patch -Np1 -i ../sysvinit-3.10-consolidated-1.patch

make

make install
```

![](https://img.164314.xyz/2024/09/f38561199d36e0adb1510675b757f997.png)

到此为止，我们已经结束了所有软件包的安装，接下来我们很快就可以进入我们的系统了!!

---

## 7.83 清理垃圾

```
rm -rf /tmp/{*,.*}
find /usr/lib /usr/libexec -name \*.la -delete
find /usr -depth -name $(uname -m)-lfs-linux-gnu\* | xargs rm -rf
userdel -r tester
```

# 8 配置系统

## 8.1 安装lfs-bootscripts

```make install```

![](https://img.164314.xyz/2024/09/72ffe016fca1527a4b001c66cb9b06b0.png)

## 8.2 配置系统主机名

```
echo "LFS" > /etc/hostname
```

## 8.3 配置 SysVinit

在linux系统进行开机的时候，会首先运行init程序，然后init程序会根据/etc/inittab文件的内容来启动系统的各种服务。在LFS中，我们使用的是SysVinit，所以我们需要配置一下。

```
cat > /etc/inittab << "EOF"
# Begin /etc/inittab
id:3:initdefault:
si::sysinit:/etc/rc.d/init.d/rc S
l0:0:wait:/etc/rc.d/init.d/rc 0
l1:S1:wait:/etc/rc.d/init.d/rc 1
l2:2:wait:/etc/rc.d/init.d/rc 2
l3:3:wait:/etc/rc.d/init.d/rc 3
l4:4:wait:/etc/rc.d/init.d/rc 4
l5:5:wait:/etc/rc.d/init.d/rc 5
l6:6:wait:/etc/rc.d/init.d/rc 6
ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now
su:S06:once:/sbin/sulogin
s1:1:respawn:/sbin/sulogin
1:2345:respawn:/sbin/agetty --noclear tty1 9600
2:2345:respawn:/sbin/agetty tty2 9600
3:2345:respawn:/sbin/agetty tty3 9600
4:2345:respawn:/sbin/agetty tty4 9600
5:2345:respawn:/sbin/agetty tty5 9600
6:2345:respawn:/sbin/agetty tty6 9600
# End /etc/inittab
EOF
```

![](https://img.164314.xyz/2024/09/b305cd889cb6ad2e21d7a36bf398fc40.png)


## 8.4 配置时区

输入hwclock如果出现的是当前时间，那么就不需要配置时区，虚拟机已经为我们准备好了，但是如果想要修改的话，可以使用下面的命令。

UTC=1表示硬件时钟是UTC时间，你也可以修改为其他值

```
cat > /etc/sysconfig/clock << "EOF"
# Begin /etc/sysconfig/clock
UTC=1
# Set this to any options you might need to give to hwclock,
# such as machine hardware clock type for Alphas.
CLOCKPARAMS=
# End /etc/sysconfig/clock
EOF
```

## 8.5 配置系统Locale
本地语⾔⽀持需要⼀些环境变量。正确设定它们可以带来以下好处：

- 程序输出被翻译成本地语⾔

- 字符被正确分类为字⺟、数字和其他类别，这对于使 bash 正确接受命令⾏中的⾮ ASCII 本地⾮英⽂字符来说是必要的

- 根据所在地区惯例排序字⺟

- 适⽤于所在地区的默认纸张尺⼨

- 正确格式化货币、时间和⽇期值

这里我将使用zh_CN.UTF-8作为我的locale，你可以根据自己的需要进行修改。(locale -a可以查看系统支持的locale)

```
LC_ALL=zh_CN.UTF-8 locale charmap
LC_ALL=zh_CN.UTF-8 locale language
LC_ALL=zh_CN.UTF-8 locale charmap
LC_ALL=zh_CN.UTF-8 locale int_curr_symbol
LC_ALL=zh_CN.UTF-8 locale int_prefix
```

再运行

```
cat > /etc/profile << "EOF"
# Begin /etc/profile
for i in $(locale); do
    unset ${i%=*}
done
if [[ "$TERM" = linux ]]; then
    export LANG=C.UTF-8
else
    export LANG=en_US.UTF-8
fi
# End /etc/profile
EOF
```

![](https://img.164314.xyz/2024/09/e2b161b62f1eb17562349b5945770dc8.png)

## 8.6 创建 /etc/inputrc ⽂件
    
```
cat > /etc/inputrc << "EOF"
# Begin /etc/inputrc
# Modified by Chris Lynn <roryo@roryo.dynup.net>
# Allow the command prompt to wrap to the next line
set horizontal-scroll-mode Off
# Enable 8-bit input
set meta-flag On
set input-meta On
# Turns off 8th bit stripping
set convert-meta Off
# Keep the 8th bit for display
set output-meta On
# none, visible or audible
set bell-style none
# All of the following map the escape sequence of the value
# contained in the 1st argument to the readline specific functions
"\eOd": backward-word
"\eOc": forward-word
# for linux console
"\e[1~": beginning-of-line
"\e[4~": end-of-line
"\e[5~": beginning-of-history
"\e[6~": end-of-history
"\e[3~": delete-char
"\e[2~": quoted-insert
# for xterm
"\eOH": beginning-of-line
"\eOF": end-of-line
# for Konsole
"\e[H": beginning-of-line
"\e[F": end-of-line
# End /etc/inputrc
EOF
```

## 8.7 创建 /etc/shells ⽂件

```
cat > /etc/shells << "EOF"
# Begin /etc/shells
/bin/sh
/bin/bash
# End /etc/shells
EOF
```

# 9 系统引导

## 9.1 创建 /etc/fstab ⽂件

```
cat > /etc/fstab << "EOF"
# Begin /etc/fstab
# 文件系统      挂载点  类型    选项       转储 检查顺序
/dev/sda3       /        ext4    defaults    1 1
/dev/sda5       swap     swap    pri=1       0 0
proc            /proc    proc    nosuid,noexec,nodev 0 0
sysfs           /sys     sysfs   nosuid,noexec,nodev 0 0
devpts          /dev/pts devpts  gid=5,mode=620 0 0
tmpfs           /run     tmpfs   defaults    0 0
devtmpfs        /dev     devtmpfs mode=0755,nosuid 0 0
tmpfs           /dev/shm tmpfs   nosuid,nodev 0 0
cgroup2         /sys/fs/cgroup cgroup2 nosuid,noexec,nodev 0 0
# End /etc/fstab
EOF
```

## 9.2 配置内核

### 9.2.1 安装内核

建议先保存快照，一步做错从头再来。

我要开始了，注意每一个字，都非常的重要

如果sources目录中含有linux-版本号文件夹，删掉，重新解压，再cd进去。
进入之后，先运行`make defconfig`, 再运行`make menuconfig`，然后再运行`make`，最后再运行`make modules_install`

这里有一个点非常的重要，因为我们的LFS是安装在虚拟机上面的，所以我们的内核配置要修改一下下。下方是需要修改的地方。此界面在运行`make menuconfig`之后会出现。

![](https://img.164314.xyz/2024/09/b2e46aef13f69e7e54cb172658d3b61b.png)

![](https://img.164314.xyz/2024/09/759dacd052fc2f1620582e2381428839.png)

![](https://img.164314.xyz/2024/09/6ba5b51115a78f873f89b8ca8b2dbbeb.png)

![](https://img.164314.xyz/2024/09/fd25cf5fa3fe0e08c59575d457479bb6.png)

![](https://img.164314.xyz/2024/09/d393bb3cbaa21e270a0ab9519ab809b1.png)

除了上方的修改之外，还需要修改下方的地方。一般来讲，只要先运行了`make defconfig`后，只需要再修改一下下方的地方就可以了。如果出现错误，建议重新设置。

- Device Drivers, Generic Driver Options, Maintain a devtmpfs filesystem to mount at /dev

- Device Drivers, Network device support, Ethernet Driver support, AMD PCnet32 PCI support

- Device Drivers, Fusion MPT device support (select the three drivers for SPI, FC, SAS)

- Device Drivers, SCSI device support, SCSI low-level drivers

- File Systems, Ext3 Journaling file system support

修改好了之后，我们就可以运行`make`了，这个过程会非常的漫长，所以请耐心等待。

运行完了之后，我们就可以运行`make modules_install`了，这个过程会非常的快，所以不用担心。再之后需要运行

```
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.10.5-lfs-12.2
cp -iv System.map /boot/System.map-6.10.5
cp -iv .config /boot/config-6.10.5
cp -r Documentation -T /usr/share/doc/linux-6.10.5
```

上方的命令是将内核的一些文件复制到了/boot目录下，这样我们就可以引导了。

### 9.2.2 Grub

之后开始修改引导文件，也就是grub


`grub-install /dev/sda` 在运行完这条命令后，会覆盖当前的引导，也就是系统在启动页面不会有宿主系统的引导了，只有LFS的引导了。

复制下方的命令，然后运行，这样就可以修改grub的引导了。如果你是跟着我一步一步来的，那么这个引导文件就是你的引导文件了。反之，你需要修改root的值。因为我的root是在sda3上的，所以我这里的root是/dev/sda3，你需要根据自己的情况来修改。
```
cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5
insmod part_gpt
insmod ext2
set root=(hd0,3)
menuentry "GNU/Linux, Linux 6.10.5-lfs-12.2" {
    linux /boot/vmlinuz-6.10.5-lfs-12.2 root=/dev/sda3 ro
}
EOF
```

一般到这步就可以直接重启了，蛮激动人心的不是吗？但是如果你想要进行一些额外的配置可以运行下方的命令。

```
echo 12.2 > /etc/lfs-release

cat > /etc/lsb-release << "EOF"
DISTRIB_ID="Linux From Scratch"
DISTRIB_RELEASE="12.2"
DISTRIB_CODENAME="Liu_Yangfeng"
DISTRIB_DESCRIPTION="Linux From Scratch"
EOF

cat > /etc/os-release << "EOF"
NAME="Linux From Scratch"
VERSION="12.2"
ID=lfs
PRETTY_NAME="Linux From Scratch 12.2"
VERSION_CODENAME="Liu_Yangfeng"
HOME_URL="https://www.linuxfromscratch.org/lfs/"
EOF
``` 

最后重启就能直接进入LFS了。用户名是root，密码是你设置的密码。

# 10 结束语

到此为止，我们已经完成了整个LFS的安装，这个过程非常的漫长，但是也非常的有趣。在这个过程中，我们学到了很多的知识，也锻炼了我们的耐心。从小就听到Linux，但是一直不知道Linux是怎么来的，现在我们通过LFS的安装，我们知道了Linux是怎么来的，也知道了Linux是怎么运行的。这个过程非常的有趣，也非常的有意义。
