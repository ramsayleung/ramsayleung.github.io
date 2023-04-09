+++
title = "大话Linux文件系统"
description = "an discussion about linux file system"
date = 2017-03-30T00:00:00+08:00
lastmod = 2022-02-23T21:41:33+08:00
tags = ["linux"]
categories = ["linux"]
draft = false
toc = true
+++

不久前，Apple 的文件系统 (Apple File System) 新推出，然后各方便一起挤身向前对APFS "评头品足",我是不了解 APFS ,所以也没有什么发言权嘛，不过话分两头；

对Linux的文件系统，我还是有了解过的，所以可以聊聊Linux 的文件系统；与Windows 和Apple 两家商业公司不同，Linux 是开源的，因此，只要你有足够的时间和能力，你就可以自己写一个文件系统，这个也是Linux 文件系统众多的主要原因。

那么有众多的Linux 文件系统，它们的差异,优缺点又是什么呢？


## <span class="section-num">1</span> File System {#file-system}

在开始 "大话" 各种文件系统的时候，我先想谈谈什么是文件系统。

一般用户平时都会有上百G 的数据，那么多的数据，应该怎么保存呢？

不知道怎么回答，可以先类比一下，想象一下你有很多书，你会怎么放置你的书籍呢？直接丢在客厅中间？或者是按书内容分类 放到书架上？如果分类的话，是按怎么划分呢？你买回来的书架一次可以放置多少书籍呢？怎么才能更快地找到你要找的书呢？

其实文件系统处理，检索文件和你放置，查找书籍是类似的！此外,在Windows 下常见到的格式化，其实就是在不同的文件系统之间切换，那么为什么数据会丢失呢？

你可以类比成你家里装修，想换新的书架，但是如果不把原来书架上的书籍搬出准备装修的屋子，装修时肯定会损坏书籍嘛！


## <span class="section-num">2</span> Journaling {#journaling}

在比较文件系统之前，先聊聊文件系统中的日志 (Journaling), 而有些文件系统是有日志功能的，而有些是没有日志功能的；

为了了解它们之间的差异，就需要先了解一下什么是日志。日志的出现本质而言都是为了更好地保存数据。

假设你正在向磁盘里写入文件，突然间断电了，但是你的文件是没有完成写入磁盘的，而你的电脑是不知道是否已经完成写入，所以下次重新来电启动电脑的时候，是没有“人”告知电脑是否要重新写数据的？这样数据就丢失了。

能不能在文件还没有完成写回磁盘又中途出现错误的时候，告诉系统你的 目标文件还有数据没有写回磁盘，你要记得完成这项工作阿？

当然可以，这就是文件系统中日志的功能，在日志的协助下，你的电脑会在日志中记着，“我要把某某文件写入到磁盘”，如果顺利完成写入磁盘工作，日志的这项记录就会被删除，如果中途出现异常，例 如断电了，重新启动的时候，你的系统就会发现，我要写入某某文件的工作还没完成，它就会继续未了的事业，这样就可以保障数据不会丢失。

(而你日常工作生活中，数据的丢失是因为你没有保存数据，即没有把数据写入到磁盘). 如图：

{{< figure src="https://www.howtogeek.com/wp-content/uploads/2017/02/img_58af43716360e.png%20" >}}

因为写入日志需要额外的工作，所以需要额外的资源，但是这样的消耗是相当值得地。


## <span class="section-num">3</span> Comparison {#comparison}

下面对比一下 各种文件系统的特性

{{< figure src="/ox-hugo/comparison_file_system.png" caption="<span class=\"figure-number\">Figure 1: </span>图来自archwiki" >}}

如果你想了解你的Linux 内核所支持的文件系统，你可以

```shell
cat /proc/filesystems
```

现在就来比较一下各种常见的Linux文件系统


### <span class="section-num">3.1</span> Ext {#ext}

Ext 是指"Extended file system(扩展文件系统)",应该算是Linux 文件系统里面的老大爷了，它是从经典的 Minix 的文件系统衍生过来的，为Linux 专门设计的，但是它缺乏很多重要的特性，比如上面提到的日志功能，所以大部份的Linux 发行版本都是不支持 Ext 了


### <span class="section-num">3.2</span> Ext2 {#ext2}

Ext2 也是不支持日志的，但是它是第一个支持扩展文件属性和2T 容量的Linux 文件系统，但是正由于Ext2 不支持日志功能，它可以更少地写磁盘，所以它适合像USB 这种闪存，但是Ext2 无法被Windows 识别的，所以它的闪存功能更多地被FAT32和exFAT所代替，换言之，Ext2 用的也不多


### <span class="section-num">3.3</span> Ext3 {#ext3}

Ext3 就是Ext2 带有日志功能的扩展，并且Ext3 也向后兼容Ext2,所以你在Ext2 和Ext3 之间切换也是不需要重新格式化滴，但是最常用的还不是Ext3,而是Ext4 :)


### <span class="section-num">3.4</span> Ext4 {#ext4}

Ext4 也是向后兼容 Ext3 和Ext2 的，所以你是可以在Ext4,Ext3,Ext2 之间切换而无需格式化文件系统。

Ext4 包含很多新的特性，例如支持存储更大的文件，支持延迟分配以 改进对闪存的支持，还能有效地减少文件的碎片化，提高利用效率。

显而易见，Ext4 是最先进的 Ext 系列的文件系统，也是大部分Linux 发行版本的默认文件系统


### <span class="section-num">3.5</span> ZFS {#zfs}

ZFS 最初是给Sun 的Solaris 设计的，Sun 被收购后，现在是属于Oracle 的，ZFS 支持 大量非常先进的特性，比如说 快照 (snapshot),动态存储 (dynamic disk striping), 驱动池等 (drive pool);

此外ZFS 文件的每个文件都是有校验和的，所以通过检查校验和，就能确定文件的完整性。但是，虽说ZFS 非常强大，却因为ZFS license 的缘故， ZFS 无法添加到 Linux 内核的。如果你非常想要尝试ZFS 的话，你可以自行添加对Linux 发行版本上面添加ZFS 的支持


### <span class="section-num">3.6</span> BrtFS {#brtfs}

Brtfs 是由Oracle 设计的一个支持写时复制 (copy on write) 的现代文件系统;

而Btrfs 的意思是 B 树文件系统 (B-Tree File System),它支持大量非常先进的特性，例如 动态inode 分配，数据校验和，有效的增量备份，驱动池，最大支持 2^64 byte 容量即16 Eib 大的文件。

BtrFS 是被设计成取代Ext 系列的文件系统的，只不过因为现在的BtrFS 还没有足够成熟，所以还没有大公司在生产环境使用 BtrFS,但是BtrFS 的未来可期


### <span class="section-num">3.7</span> JFS {#jfs}

JFS 是IBM 为IBM 自家的AIX 操作系统设计的日志文件系统 (Journaled File System), 后来迁移到了Linux 系统上 (HP-UX 也有一个叫做JFS 的文件系统).

在AIX 系统上是存在过两代的JFS文件系统的，分别是 JFS1和JFS2,而Linux 上就只有JFS2了(Linux 上的JFS都是指JFS2)。

JFS 无论在处理大文件还是小文件都有非常不错的表现，并且CPU 占用也是比较低的；JFS 也是支持非常多的特性的，例如 B+ 树，动态Inode 分配，并发IO等。

JFS 也是一个设计优秀的文件系统并且支持大部分的Linux 发行版本，但是因为它最初是为AIX 设计，所以在处于生产环境上的Linux服务器的测试就不如Ext :(.


### <span class="section-num">3.8</span> ReiserFS {#reiserfs}

ReiserFs 是第一个被引进Linux 标准内核的日志文件系统(在内核版本为2.4.1的时候引进),也是Linux 文件系统的一次飞跃，那时的ReiserFS 包含了很多Ext 没有的新特性。

虽说 ReiserFS 在Linux 有一个非常华丽的开头，但是后来ReiserFS 的开发就陷入了停滞，因为ReiserFS 的核心开发者Hans Reiser(ReiserFS 名字的来由)因为谋杀妻子而被收监 :(。

而后来的ReiserFS 也没有出现在主要的Linux 内核版本里面，虽说ReiserFS是非常好的文件系统，但是它前景如何，我们也只能拭目以待了


### <span class="section-num">3.9</span> XFS {#xfs}

XFS 最初是Silicon Graphics 为SGI IRX 操作系统设计的64位高性能文件系统，在2001 年迁移到了Linux.

得益于XFS 基于 allocation groups 的设计，XFS 拥有非常优秀的并行IO 能力，并支持延迟分配 (delayed allocation) 以改进文件碎片化，在某种程度上，XFS 和Ext 有一定的相似；

此外，虽说XFS 是高性能的文件系统，但是那只是针对大文件而言的，对于小文件XFS 就有点力所不能及 (当然，这是相对而言).

所以如果是需要 经常处理大文件的服务器，XFS 会是一个很好的选择


## <span class="section-num">4</span> 小结 {#小结}

如果将Linux 的文件系统进行分类的话，还可以分成 FUSE (Filesystem in Userspace,
让用户在没有权限的情况下，创建自己的文件系统), Stackable file System (先进的多层次统一文件系统), Read-only file systems (只读文件系统), Clustered file systems
(集群文件系统)等等。

Linux 的文件系统还有很多，每一种都有自己的特点；但是如果你想问："Linux 最好的文件系统是什么？";这个问题就跟 "最好的Linux 发行版本是什么？", "最好的文本编辑器是什么？" 一样，是没有标准答案，一千个人都有一千个哈姆雷特了，你的哈姆雷特是什么样子，只有你自己清楚。

不同的文件系统对应不同的场景，只有针对特定场景的最优解决方案！如果还不知道怎么选择，那就选择Ext4 吧，无法做出选 择时，默认的就是最好 :)


## <span class="section-num">5</span> 参考 {#参考}

-   <https://en.wikipedia.org/wiki/File_system>
-   <https://btrfs.wiki.kernel.org/index.php/Main_Page>
-   <https://en.wikipedia.org/wiki/JFS_(file_system)>
-   <https://en.wikipedia.org/wiki/ReiserFS>
-   <https://www.howtogeek.com/howto/33552/htg-explains-which-linux-file-system-should-you-choose/>
-   <https://en.wikipedia.org/wiki/ZFS>
-   <https://en.wikipedia.org/wiki/XFS>
-   <https://wiki.archlinux.org/index.php/file_systems>
