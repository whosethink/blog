+++
title = "Arch/Linux 安装 (1)"
date = 2020-06-10

[taxonomies]
tags = ["arch"]
+++

### 此情此景

在此之前, 我一直是 Ubunut 用户, 16.04 -> 18.04 -> 20.04. 随着时间的堆积, 一丝丝隐隐的不安慢慢在我心底弥散. 仿佛屋宅的墙壁在褪色, 照片在变黄的感觉, 我需要一些些的改变. 走上小船, 解开缆绳, 漂吧.

<!-- more -->

### 参考及说明

这篇文章的安装步骤是针对我自身的情况, 有一些预先的前提. 更详细的安装教程可参阅

+ [以官方Wiki的方式安装ArchLinux](https://www.viseator.com/2017/05/17/arch_install/)
+ [ArckWiki - Installation guide](https://wiki.archlinux.org/index.php/installation_guide)

我的本子是 HP Zhan66 AMD 4700U, 引导方式是 EFI, 网络环境是 wifi, 需要安装的磁盘为 /dev/nvm0n1.

### Arch 安装
1. 连接网络(wifi)

    输入 ```wifi-menu```, 选择需要连接的 wifi, 输入配置文件名称(此处提供了默认名称), 接下来就是输入 wifi 密码. 如果一切顺利, 终端没有指示出错误, oh yes!

    在连接 wifi 的过程中, 我遇到一些小问题, 记录在文末. 如果你也不幸遇到了, 希望对你有所帮助, 毕竟连不上网络, 那多急人哎.

    ```sh
    # 网络测试
    ping github.com
    ```

2. 校准时钟

    使用 ntp 网络时间服务来校准时钟

    ```sh
    timedatectl set-ntp true
    ```

    查看时间是否正确

    ```
    # 输出结果的第一行为本地时间
    timedatectl status
    ```

3. 磁盘分区

    看看系统下有哪些磁盘, 目标磁盘在其中否

    ```sh
    fdisk -l
    ```

    磁盘分区, 我要安装 Arch 的磁盘为 /dev/nvm0n1, 输入以下命令

    ```sh
    fdisk /dev/nvm0n1
    ```

    接下来在 fdisk 下输入命令:

    1. 创建 GPT 分区表, 输入命令: g

    2. 创建 EFI 分区, 依次输入命令: n, Enter, +1G, t, 1, 1

    3. 创建 root 分区: 依次输入命令: n, Enter, Enter

    4. fdisk -l 检查分区是否正确

    5. 保存, 输出命令: w


    __命令解释(以创建 EFI 分区为例):__

    1. n: 创建一个新的分区

    2. Enter: 直接回车, 表示使用默认值

    3. +1G: 分区大小为 1G

    4. t: 设置分区类型

    5. 1: 为第一个分区设置类型

    6. 1: 类型为 1 (分区类型可通过 l 命令查看)

4. 格式化分区

    __通过 fdisk -l 查看要格式化的分区__

    1. 第一个分区(EFI 分区)格式化为 FAT32 格式:

    ```sh
    mkfs.fat -F32 /dev/nvm0n1p1
    ```

    2. 第二个分区格式化为 EXT4 格式:

    ```sh
    mkfs.ext4 /dev/nvm0n1p2
    ```

5. 挂载分区

    1. 创建 EFI 分区挂载点

    ```sh
    mkdir /mnt/boot
    ```

    2. 挂载 EFI 分区

    ```sh
    mount /dev/nvm0n1p1 /mnt/boot
    ```

    3. 挂载 root 分区

    ```sh
    mount /dev/nvm0n1p2 /mnt
    ````

6. 安装基础包

    1. 修改软件安装源(默认配置在国内比较慢)

    ```sh
    # 备份默认配置
    mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak

    # 设置安装源
    echo 'Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch' > /etc/pacman.d/mirrirlist
    ```

    2. 安装软件包到磁盘

    ```sh
    pacstrap /mnt base base-devel linux linux-firmware
    ```
7. 生成 fstab 文件

    ```sh
    genfstab -L /mnt >> /mnt/etc/fstab
    ```

    命令 cat /mnt/etc/fstab 查看 fstab 文件是否正确. 我在这一步时遇到了问题, 记录在文末.

8. 更改 root 目录

    ```sh
    arch-chroot /mnt
    ```

    第 5 步中, 我将 /dev/nvm0n1p2 挂载到了 /mnt 下, 而 /dev/nvm0n1p2 就是 Arch 安装后的 root 目录. 进行完这一步后, 后续的操作就相当于在安装好后的 Arch 里操作, 也就是所做的操作会在之后的系统里生效. 

   特别提一下这一步, 因为在之后我遇到过一次问题, boot 引导无法完成, 进不去终端操作. 是使用 liveOs, arch-chroot进去救砖的~~~. 会记录在后续图形界面安装文里.

9. 设置时区

    不清楚的话, 搜索一下该设置成哪个时区. ```timedatectl list-timezones``` 可以罗列出各时区.

    ```sh
    # 设置时区为上海
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    # 把时间信息写入硬件
    hwclock --systohc
    ```

10. 安装基础软件

    可以直接使用 ```pacman -S``` 来安装了, 一路回车.

    ```sh
    pacman -S vim dialog wpa_supplicant ntfs-3g networkmanager netctl
    ```

11. 设置语言环境

    ```sh
    # 备份语言配置文件
    mv /etc/locale.gen /etc/locale.gen.bak

    # 设置语言环境
    echo "zh_CN.UTF-8 UTF-8\nzh_HK.UTF-8 UTF-8\nzh_TW.UTF-8 UTF-8\nen_US.UTF-8 UTF-8" > /etc/locale.gen

    # 生成 locale
    locale-gen
    ```

11. 修改 root 密码

    ```sh
    passwd
    ```

    会两次要求输入新密码, 密码不会在屏幕上显示, 输入完回车就行.

12. 生成引导项

    ```sh
    # 安装软件
    pacman -S os-prober grub efibootmgr

    # 部署 grub
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub

    # 生成引导配置
    grub-mkconfig -o /boot/grub/grub.cfg
    ```

    使用 ```vim /boot/grub/grub.cfg``` 查看配置项是否正确. 可搜索 menuentry, 看看有没有 Arch 和其他系统(如果有的话)的配置.

13. 重启

    ```sh
    # 退出到 LiveOs 环境
    exit

    # 卸载 EFI 分区
    umount /mnt/boot

    # 卸载 root 分区
    umount /mnt
    # 重启
    reboot //重启
    ```

### 问题记录
+ wifi-menu 连接 wifi 失败

    1. 缺少配置文件

        /etc/netctl/ 目录下没有相关 wifi 的配置文件.

        ```sh
        # 复制 examples 下的配置文件, 修改相关项即可. 我这边是 wpa 的 wifi
        cp /etc/netctl/examples/wireless-wpa /etc/netctl/MyWifi
        ```

        然后修改 /etc/netctl/MyWifi 文件中的 ESSID(wifi 名称) 和 Key(wifi 密码)

    2. 网卡被占用

        ```sh
        # 如果网卡是 wlp2s0
        ip link set wlp2s0 down
        ```

        ```ip link``` 可以列出当前的网卡. 无线网卡通常是 w...

    3. 网络配置文件名称

        出现过一次, 不是很确定... 默认的配置文件名会是 wlp2s0-wifiname 这样, 然后这个配置文件就创建失败. 去掉前缀, 使用 wifiname 作为配置名创建成功.

+ fstab 缺少 boot 分区信息
    安装过程第 7 步中, 查看时缺少 EFI 分区的信息, 原因是 EFI 分区不知道去哪了.
    ```sh
    # 删除无效的 fstab 文件
    rm /mnt/etc/fstab
    # 重新挂载 EFI 分区
    mount /dev/nvm0n1p1 /mnt/boot
    # 重新生成 fstab 文件
    genfstab -L /mnt >> /mnt/etc/fstab
    ```

---
那天, 轻风浮动, 空气中传递着阵阵的浮躁; 那天, 阳光倾洒, 光热中弥散着隐隐的焦虑.
