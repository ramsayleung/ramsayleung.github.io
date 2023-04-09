+++
title = "枯树逢春之ArchLinux领风骚"
description = "An description about how to install arch linux in an old machine"
date = 2017-02-27T00:00:00+08:00
lastmod = 2022-02-23T21:07:14+08:00
tags = ["linux"]
categories = ["linux"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 枯树 {#枯树}

周末回了一趟家，没带自己的笔记本，在家闲来无事，无意中看到墙角的电脑，已经尘封已久反正无事，何不玩玩这台老古董呢？于是把电脑拿去修理店把坏了的硬件修好。

离开店的时候，老板说：“你的系统有问题，我看到你自己也有Ghost,就不帮搞这系统了， 你自己都能解决的，推荐你还是用XP吧，这电脑配置低，还是XP好用”。我忍不住回头对老板一笑 :)


## <span class="section-num">2</span> 春至 {#春至}

像我这种Linuxer,这么可能再装回XP呢，最初装Win7,也是考虑到老爹的技术 hold 不住 Linux, 现在手机那么发达，他就不需要电脑了，所以，此时不装Linux,更待何时呢？


## <span class="section-num">3</span> Arch Linux {#arch-linux}

我没有选择 Xubuntu 这种适合老机器的 Ubuntu 衍生发行版本，因为我不喜欢Ubuntu, 所以我最后选择的是 Arch linux,官网说最低配置只需500MB内存，800MB的 硬盘存储空间，正适合家里的老家伙


### <span class="section-num">3.1</span> 安装过程 {#安装过程}


#### <span class="section-num">3.1.1</span> 下载镜像 {#下载镜像}

[Download Link](https://www.archlinux.org/download/) ,在网易的镜像下载ISO, 然后用dd刻录到U盘，Windows 可以选择 [USBwriter](http://sourceforge.net/projects/usbwriter)


#### <span class="section-num">3.1.2</span> 分区 {#分区}

使用fdisk, 我的硬盘是/dev/sda,如果还有一块硬盘，那应该就是/dev/sdb

```shell
fdisk /dev/sda
```

-   n:新建一个分区，p 指主分区，e 是指扩展分区(逻辑分区是建立在扩展分区上的)
    一块硬盘主分区加上扩展分区最多只能是4个
-   d: 删除
-   m: 查询其他命令，不知道怎么操作就输入m 吧

分区结束以后，输入 `w` 完成分区 (我分了三个分区 /dev/sda1 -&gt; swap
/dev/sda2 -&gt; / /dev/sda3 -&gt; /home)


#### <span class="section-num">3.1.3</span> 格式化分区 {#格式化分区}

格式化 sda2 sda3为ext4格式：

```shell
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
```

格式化sda1 为swap(虚拟内存),一般是内存的两倍，当然如果你的内存很大的话就不用划这个分区了

```shell
mkswap /dev/sda1
```

激活swap

```shell
swapon /dev/sda1
```


#### <span class="section-num">3.1.4</span> 挂载 {#挂载}

将sda2挂载到/mnt,其实就是让sda2分区做系统的根分区，/mnt/home同理

```shell
mount /dev/sda2 /mnt
mount /dev/sda3 /mnt/home
```


#### <span class="section-num">3.1.5</span> 更新pacman源 {#更新pacman源}

网易的源不错，编辑 `/etc/pacman.d/mirrorlist` 添加 `Server = http://mirrors.163.com/archlinux/$repo/os/$arch`

```shell
vim /etc/pacman.d/mirrorlist
```

然后添加；添加完之后，更新一下

```shell
pacman -Syy
```


#### <span class="section-num">3.1.6</span> 安装基本系统 {#安装基本系统}

安装基本系统到 /mnt,即sda2分区

```shell
pacstrap /mnt base base-devel
```

需要安装的都安装吧，然后走开煮一杯咖啡，慢慢品尝


#### <span class="section-num">3.1.7</span> 生成fstab {#生成fstab}

fstab 的作用：

> The fstab(5) file can be used to define how disk partitions, various other block devices, or remote filesystems should be mounted into the filesystem

生成fstab,并且查看是否正确生成fstab

```shell
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```


#### <span class="section-num">3.1.8</span> 配置系统 {#配置系统}

切换到新的系统，然后你会发现命令行提示符发生了改变

```nil
arch-chroot /mnt
```

<!--list-separator-->

1.  设置地区

    ```shell
    ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    ```

<!--list-separator-->

2.  设置语言

    编辑 `/etc/locale.gen`,因为该文件所有的信息都是被注释滴，所以在最上面添加`en_US.UTF-8 UTF-8` 即可

    ```shell
    vim /etc/locale.gen
    ```

    然后添加；添加完成后，执行 `locale-gen`

    ```shell
    locale-gen
    ```

    接着配置 `locale.conf`

    ```shell
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    export LANG=en_US.UTF-8
    ```

<!--list-separator-->

3.  设置主机名

    ```shell
    echo samray-arch > /etc/hostname
    ```

<!--list-separator-->

4.  设置密码

    ```shell
    passwd
    ```

<!--list-separator-->

5.  配置网络

    ```shell
    pacman -S net-tools
    systemctl enable dhcpcd.service
    ```

<!--list-separator-->

6.  安装GRUB

    ```shell
    pacman -S grub-bios
    ```

    把grub 安装到硬盘sda,如果双系统的话，还要视情况做更改

    ```shell
    grub-install --recheck /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg
    ```


#### <span class="section-num">3.1.9</span> 收尾工作 {#收尾工作}

```shell
exit
umount /mnt/home
umount /mnt
reboot
```

这样Arch linux 就装好了，不过你重启会发现，你的系统是没有图形化界面的


### <span class="section-num">3.2</span> 安装桌面环境 {#安装桌面环境}


#### <span class="section-num">3.2.1</span> 安装x服务 {#安装x服务}

```shell
pacman -S xorg-server xorg-server-utils xorg-xinit
```


#### <span class="section-num">3.2.2</span> 安装显卡驱动 {#安装显卡驱动}

查找自己的显卡类型

```shell
ispci |grep VGA
```

然后搜索匹配自己显卡的驱动

```shell
pacman -Ss xf86-video |less
```

Intel 集成显卡：

```shell
pacman -S xf86-video-intel
```

虚拟机显卡：

```shell
pacman -S xf86-video-vesa
```

笔记本触摸板驱动 (老家伙是台式，不需要了):

```shell
pacman -S xf86-input-synaptics
```

安装输入法

```shell
pacman -S scim-pinyin
```

先安装 slim(图像登录管理器)

```shell
pacman -S slim
```

安装xfce4

```shell
pacman -S xfce4
```

启动xfce4

```shell
startxfce4
```

基本就大功告成了，因为我的台式电脑是bios, 所以不用折腾uefi, 还有无线网络。

**Action is louder than words**,还是多动手才行，我都装了三次才成功，内核空指针和段错误都遇到了 :）


### <span class="section-num">3.3</span> 参考 {#参考}

<https://wiki.archlinux.org/index.php/installation_guide>
