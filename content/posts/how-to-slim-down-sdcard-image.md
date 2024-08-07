+++
title = '如何对一个SD CARD的镜像进行瘦身'
date = 2024-08-02T17:38:10+08:00
categories = "编程经验"
tags = ['raspberrypi', "image", "slim down"]
summary = '本文介绍了如果制作一个树莓派或类似产品的系统镜像，并对其进行瘦身，以便在更小尺寸SDCARD上使用。'
+++

# 起因

当我们制作了一个树莓派，或者其他类似产品（如香橙派，R2S 等）的系统，并配置完成之后，为了灾备考虑，或者方便将配置好的系统快速分发，我们通常会做一个镜像。

实际使用时，可能我们会使用较大容量的 SDCARD，但是这样做出来的镜像也会和 SDCARD 的容量一样大，比如 16G 甚至 32G，而实际上刚刚做好的系统，实际可能占用空间才 2，3G 大小。那么这样就会出现两个问题：

1.  分发不利，动辄 16G 甚至 32G，网上传送很耗时间。
1.  无法烧录到小容量的 SDCARD 上，如果上面的传输问题，还可以使用压缩工具有效解决，但是镜像的大小决定了它能烧录的最小容量，比如 16G 的镜像，只能烧录到至少 16G 大小的 SDCARD，即使系统实际只占用 3G，也无法烧录到 4G 或者 8G 的 SDCARD 中。

为了解决以上问题，我们提供以下解决方案。

# 解决方案

## 镜像提取

在 Linux 环境下，其实将 SDCARD 的内容提取为一个镜像非常容易，只需先找到该镜像所属的设备文件即可，通常为`/dev/sdc`这类的。

无法确定到底是哪个时，可以建立一个目录，然后使用`mount`命令，将设备加载到该目录，再进入该目录，根据内容确认是否为该设备。注意，`/dev/sdc`只是该设备，也就是 SDCARD，实际内容在 SD 卡的某个分区上，所以`mount`时需要指定特定分区，比如新建的目录为`/test`，假设`/dev/sdc`是目标 SD 卡，那么命令就应该是类似于`sudo mount /dev/sdc1 /test`这样的，其中`/dev/sdc1`就是 sdc 上的第一个分区，如果有多个分区可能会有 2，3，4 等数字。

当然，其实还有一个简单的方法，就是在通过插拔 SDCARD 读卡器，根据`/dev`目录下相应的设备文件的有无来判断那个文件夹代表该 SDCARD。

当确定好哪个设备是目标 SD 卡后，使用`dd`命令，将该设备作为数据源，即可将整个 SDCARD 的内容都写到某个文件中。**这里需要注意的是**，如果之前已经 mount 到某个目录了，那么需要先使用`umount`命令进行卸载，如`sudo umount /test`，然后再执行`dd命令`：

```shell
sudo dd if=/dev/sdc of=~/sd-card-copy.img bs=1M status=progress
```

如此一来，就会将 SD 卡的内容写到 HOME 目录下的`sd-card-copy.img`文件中了。其中的`bs=1M`是指定每次读写都是 1M，这样可以加快速度（你也可以适当调整提高效率，如 4M），`status=progress`是在操作时显示进度信息。

## 镜像瘦身

如**起因**一章所说，单单只做镜像提取操作，生成的文件大小与 SD 卡的容量一致，显然太大了。所以我们还要将镜像瘦身到实际大小，方便分发，也使得小容量 SDCARD 也能烧录。
其实网上可以找到很多文章，介绍如何对 SDCARD 进行瘦身，但是其实这会造成另一个比较大的问题，那就是瘦身完成后的镜像，你再烧录到新的 SDCARD 后，会发现你的文件系统的容量就是你的镜像大小，其他的容量都是未分配状态。
这又使得你必须在烧录好 SDCARD 后，然后再对这个卡进行扩容，使得可以使用整张卡的容量。
整个过程非常复杂，且每备份复原一次，都要重复镜像瘦身，然后再对 SDCARD 扩容，非常繁琐。
幸好已经有大神做好了脚本，直接使用即可。

从 https://github.com/Drewsif/PiShrink 下载 PiShrink 这个脚本，然后执行即可。它支持以下功能：

1.  对目标镜像瘦身
1.  当烧录好之后，在第一次启动时，可以自动扩容到整张 SD 卡的容量
1.  可以将瘦身后的镜像再进行压缩 ，格式可以选择 gzip 或 xz
1.  压缩时可以选择多核并行压缩，提高压缩效率
1.  可以选择在瘦身的同时，删除系统日志, apt archives,DHCP 租赁信息和 ssh hostkeys，减少关键信息泄露的风险
1.  可以对启动失败的镜像，尝试高级修复

使用方法也很简单

### 安装 PiShrink

```shell
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/bin
```

### 使用 PiShrink

```shell
sudo pishrink.sh ~/sd-card-copy.img
```

可以提供以下参数

```plaintext
  -s         在该镜像启动后不自动扩容文件系统
  -v         显示详细信息
  -r         如果正常启动失败，启用高级修复工具尝试修复
  -z         镜像瘦身后使用gzip进行压缩
  -Z         镜像瘦身后使用xz进行压缩
  -a         使用多核进行并行压缩
  -p         删除系统日志, apt archives, dhcp租赁文件和ssh hostkeys
  -d         将debug信息写入日志文件
```

一般会用到的是`-p`或者其他一些压缩相关参数。

# 烧录镜像

镜像烧录在 Linux 下也很简单，还是使用`dd`命令，只是将`if`和`of`进行翻转即可。当然，也需要事先确认那个设备文件是 SDCARD。

```shell
dd if=~/sd-card-copy.img of=/dev/sdc bs=1M status=progress
```

然后再将该 SDCARD 放入目标设备，看看是否能正常启动即可。除非在对镜像瘦身时设定了`-s`参数，否则第一次启动由于需要对文件系统扩容，会相对慢一点儿，需要稍微等待一会儿。
如果一切 OK，那么就说明 SDCARD 烧录正常了。
