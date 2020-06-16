+++
title = "Arch/Linux 安装 (2)"
date = 2020-06-12

[taxonomies]
tags = ["arch"]
+++

### 情景

上一篇中安装好了 Arch, 作为日常使用, 图形界面还是要的. 虽然在那年, 仅仅看到了粗糙的黑框, 加上跳动的字符的刹那, 心已向往之. 这一篇中, 介绍桌面环境的安装, 以及一些美化选项.

<!-- more -->

更多的可以参阅这篇文档:
1. [ArchLinux安装后的必须配置与图形界面安装教程](https://www.viseator.com/2017/05/19/arch_setup/)

### 桌面环境

1. 连接网络(同上一篇中第一步)
    ```sh
    wifi-menu
    ```

2. 显卡驱动
    ```sh
    # ADM GPU
    pacman -S xf86-video-amdgpu
    ```
    __具体安装哪个显卡驱动, 参考 [ArchWiki - Xorg (Driver installation 一节)](https://wiki.archlinux.org/index.php/Xorg).__

3. 桌面服务 - Xorg

    ```sh
    pacman -S xorg
    ```

4. 桌面环境 - KDE
    
    ```sh
    pacman -S plasma
    ```

    __在此没有直接安装 KDE 的常用软件(终端, 文件管理器等), 如果需要可通过```pacmac -S kde-applications```安装.__

5. 桌面管理器 - sddm

    ```sh
    pacman -S sddm
    # 启用 sddm
    systemctl enable sddm
    ```

    使用 sddm 的过程中, 出现过一次 boot 无法完成的情况, 记录在文末.

6. 网络配置

    ```sh
    systemctl disable netctl
    systemctl enable NetworkManaget
    pacman -S network-manager-applet
    ```

7. 创建普通用户

    KDE 登录界面, 默认不包含 root 用户, 一般情况也不使用 root 操作系统. 在此创建一个普通用户.

    ```sh
    # USERNAME 修改为你的用户名
    useradd -m USERNAME
    # 设置密码
    passwd USERNAME
    ```

    普通用户默认无法通过 sudo 执行命令. 可以像参考文档里一样, 把用户加入到 wheel 组, 然后赋予 wheel 组 sudo 权限. 也直接赋予用户 sudo 权限, 在 ```/etc/sudoers``` 里添加 ```USERNAME ALL =(ALL) ALL``` (USERNAME 改成具体的用户名)一行即可.

8. 重启

    ```sh
    reboot
    ```

### 美化

1. 壁纸 [wallhaven.cc](https://wallhaven.cc/)

    + 有以图搜图的功能
    + 可以选择指定分辨率(精确或者模糊)
    + 标签分类
    + 直接选择
    + 下载速度可能有点不尽人意

2. 主题/风格/图标

    在 System Setting 里直接设置. 设置界面右下角有下载选项, 可以在里面浏览选择你中意的类型, 然后点击安装应用即可.

    我使用的主题是 sweet, 风格是 sweet, 图标是 [candy-icons](https://github.com/EliverLara/candy-icons). 如果觉得整体的色调有些晃眼, 可以在 System Setting -> Colors 下选择一个低调一些的颜色配置, 比如 Breeze Dark.

3. 任务栏

    任务栏的基本样式(显示图标, 透明度等)是跟着上面的主题样式走的. 主要有如下的可配置:

    + 任务栏位置
    + 任务栏的大小(长度, 高度)
    + 可见性(可自动隐藏)
    + 各组件的位置
    + 显示哪些组件
    + 组件本身又会提供一些可配置项

### 问题
1. boot 引导无法完成(在另一台机子上遇到)

    boot 引导过程中, 卡在了 ```[OK] Reached target Graphical Interface``` (grub 开启 verbose 模式)处, 无法进入终端.

    怀疑是显卡驱动的原因(有 N 卡), 禁用-卸载显卡驱动, 无果(一筹莫展);


    怀疑是什么什么配置出了问题, 恢复原配置后, 无果(心如死灰); 

    chroot 进系统, 在日志文件中找到了一丝蛛丝马迹, sddm(莫不是 TA ?);

    ```systemctl disable sddm```, 重启, 可进入终端界面(心中暗喜);

    Google 寻之, [sddm/issues/605](https://github.com/sddm/sddm/issues/605), 可能是 sddm 的 bug, 重新安装 xorg-server, 久别重逢(笑逐颜开).

2. 屏幕亮度无法调节(AMD 4700U)

    亮度调无法调节屏幕亮度, 始终是最亮的.

    调整内核参数, 无果;

    重置显卡驱动, 无果;

    使用其他的亮度调节工具, 无果;

    内核升级到 5.7 版本, 无果;

    寻寻觅觅, 终于 [change brightness](https://askubuntu.com/questions/62249/how-do-you-change-brightness-color-and-sharpness-from-command-line)

    ```sh
    # 修改最后的数字来调节亮度
    xrandr --output `xrandr --current | grep ' connected' | cut -f1 -d " "` --brightness 0.5
    ```

    虽然不是很美好, 但比起要在大半夜凝望最亮的屏, 呜.
