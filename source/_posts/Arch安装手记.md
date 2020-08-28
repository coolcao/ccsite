---
title: Arch安装手记.md
date: 2020-08-28 11:57:28
tags: [arch, arch安装]
categories:
- 技术博客
- 原创
---

# arch安装手记
其实arch的安装并不复杂，如果你使用图形化安装工具安装过其他linux发行版，那么你应该知道安装时会进行一系列的设置，设置分区，设置用户名等等。
arch的安装没有图形化的工具，因此需要使用命令行来设置。其设置的内容也和图形化安装并无任何差异。
下面整理了一下安装一个arch所基本的命令。

<!-- more -->

## 准备
### 下载镜像
去arch的官网下载一个安装镜像。
官方建议使用BitTorrent：
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586707_20200828114516652_641384557.png)

也可以找到中国的镜像源，使用http下载：
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586712_20200828114610959_1529825206.png)

### 烧录启动U盘
使用dd命令烧录启动U盘：
```
sudo dd if=/arch.iso of=/dev/disk2 bs=1m
```

## U盘启动
### 设置网络

```
iwctl
```

iwctl工具设置无线网络。连接到网络。


### 更新系统时间

```
 timedatectl set-ntp true
```

## 磁盘分区
查看磁盘
```
fdisk -l
```
使用fdisk命令，进入对应磁盘进行设置：

```
fdisk /dev/sda
```

### 创建GPT分区表：

```
g
```

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586689_20200827174200222_340869473.png)

### 创建分区：

UEFI 与 GPT

|挂载点	|分区	|分区类型	|建议大小|
|----|----|----|----|
|/mnt/boot 或 /mnt/efi	|/dev/sdX1	|EFI 系统分区|	260–512 MiB|
|/mnt	|/dev/sdX2	|Linux x86-64 根目录 (/)	|剩余空间|
|[SWAP]	|/dev/sdX3	|Linux swap (交换空间)	|大于 512 MiB|

以上为Arch官方wiki建议的分区，最少两个区，一个启动分区，挂载到/mnt/boot，一个系统根分区，挂载到 /mnt。交换分区可选。

这里我们三个分区都创建了。

#### 创建EFI启动分区：

```
n
```
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586685_20200827174127363_1691353716.png)
此时要求输入分区号，这里我们什么都不用输，直接回车，选择默认即可。下面根分区和交换分区同理。
然后会要求输入分区大小，我们这里使用 `+256M` 指定分区大小为 256M 。

修改EFI分区类型：
新建的分区默认为Linux FileSystem类型，我们需要修改为EFI类型：

```
t
```
然后选择1。1为EFI类型，如果要查看所有类型，可键入L查看所有类型，然后使用q退出查看。
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586690_20200827174522991_1654346162.png)

#### 新建交换分区:

```
n
```

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586692_20200827174736451_1125167025.png)

交换分区，建议为内存两倍，我虚拟机内存设置1G，所以这里交换分区为2G。

然后修改交换分区类型：

```
t
```
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586693_20200827174911875_2088677340.png)
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586695_20200827175116277_962828601.png)

通过查看类型，我们知道，交换分区类型为19:
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586694_20200827175032732_1632359518.png)

#### 创建根分区
最后，将剩下的所有空间，创建根分区：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586695_20200827175223950_971016862.png)

这样，就创建好了三个分区。
我们可以通过 `p` 命令来打印刚才创建的分区表，类似以下正常：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586696_20200827175342047_1152166081.png)

#### 保存并退出fdisk

检查无误后，我们就可以保存分区并退出fdisk了。

```
w
```

### 格式化分区并挂载
#### 1. 格式化EFI分区
```
mkfs.fat -F32 /dev/sda1
```

#### 2. 格式化根分区
```
mkfs.ext4 /dev/sda3
```

#### 3. 格式化交换分区
```
mkswap /dev/sda2
swapon /dev/sda2
```

#### 4. 挂载分区

```
mount /dev/sda3 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```
然后通过df命令查看挂载，输出如下证明ok。

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586702_20200828100654576_30871350.png)
## 安装系统
### 设置软件源
我们更新软件源为国内镜像，这样可以加快安装速度。
```
curl -L -o /etc/pacman.d/mirrorlist "https://www.archlinux.org/mirrorlist/?country=CN"
```
然后编辑 /etc/pacman.d/mirrorlist 文件，将China下面的软件源前面#去掉即可。最后如下：

![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586696_20200827182129160_1864703250.png)

### 安装基础的软件包
```
pacstrap /mnt base linux linux-firmware amd-ucode intel-ucode bash-completion vim networkmanager pacman-contrib sudo
```

其中`base`，`linux`，`linux-firmware`是必装的，`amd-ucode`和`intel-ucode`看你的处理器是AMD的还是英特尔的选一个。
剩下的就是一些linux常用的应用了，随自己需要安装。


安装完成后，生成fstab文件：

```
genfstab -U /mnt >> /mnt/etc/fstab
```

最后切换到新安装的系统：

```
arch-chroot /mnt
```

## 新系统的基础设置
### 设置时区
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

### 设置主机名
```
vim /etc/hostname
```

```
ThinkPad
```

设置hosts
```
vim /etc/hosts
```

```
127.0.0.1 localhost 
127.0.0.1 ${hostname}
::1 localhost ip6-localhost ip6-loopback
::1 ${hostname}
```
${hostname}为上面设置的主机名

### 网络设置
```
pacman -S dhcpcd
```

```
systemctl enable dhcpcd
```


### 本地化

```
vim /etc/locale.gen
```

留一个英文和简体繁体中文三个即可。
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```

生成本地化配置文件:
```
local-gen
```

创建locale.conf，编辑内容如下：
```
vim /etc/locale.conf
```

这里只留英文，因为还没图形化界面，在终端如果使用中文会导致乱码。
```
LANG=en_US.UTF-8
```

### 用户管理

#### 设置root密码
```
passwd
```

#### 添加新用户
添加新的个人用户:
```
useradd -m -G wheel -s /bin/bash ${username}
passwd ${username}
```
新添加的用户无法使用sudo命令，我们将其添加到sudoers:

```
vim /etc/sudoers.d/${username}
```
这个文件根据刚才新添加的用户名新建，内容为：

```
${username} ALL=(ALL) ALL
```

### 安装grub

安装grub软件包：
```
pacman -S grub
```

安装grub至esp分区：
```
grub-install --removable --target=x86_64-efi --efi-directory=/boot
```

生成grub配置文件：
```
grub-mkconfig -o /boot/grub/grub.cfg
```

安装完成后会有如下显示，说明安装成功。
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586697_20200827185248765_1838662079.png)

## 退出chroot环境并重启
到这里，系统的基础设置就完成了，我们现在可以退出chroot环境并重启。
### 退出chroot
```
exit
```

### 重启
1. 卸载分区
umount -R /mnt
2. 重启
reboot
![](https://img-vnote-1251075307.cos.ap-beijing.myqcloud.com/1598586704_20200828103052843_2090918295.png)
重启后，当看到如上登录界面时，就已经安装成功了。

接下来，就是自由发挥的时间了。