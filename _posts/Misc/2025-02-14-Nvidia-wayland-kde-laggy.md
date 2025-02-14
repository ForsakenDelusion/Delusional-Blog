---
title: 解决Nvidia显卡在Wayland下使用KDE卡顿的现象
date: 2025-02-14 13:41:45 +08:00
filename: 2025-02-14-Nvidia-wayland-kde-laggy
categories:
  - Misc
tags:
  - Arch
  - Linux
  - Environment
dir: Misc
share: true
---
本人桌面环境

RTX 3060 Laptop
Wayland + KDE 
Linux-zen Nvidia-dkms

一些可供参考的链接。

https://discuss.kde.org/t/wayland-plasma-6-1-nvidia-555-intermittent-application-stuttering/17629

https://www.reddit.com/r/kde/comments/qr1obx/laggy_nvidiaarchwaylandplasma_any_way_to_improve/

https://discuss.kde.org/t/kde-plasma-wayland-nvidia-low-fps-stutters/27064

https://bbs.archlinux.org/viewtopic.php?pid=2182638

## 方法一

非常朴素的办法，就是开启`OBS`，根据论坛老哥们的测试，卡顿是由GPU莫名其妙被限制频率引起，但是打开`OBS`貌似会让显卡不降频率。

## 方法二

手动限制最低频率，缺点是可能会加大功耗。

>That’s all speculation, but I am 99% sure that, as Nate said above, it’s all down to clock speeds. On the new gaming desktop, running the command `sudo nvidia-smi --lock-gpu-clocks=1980,3105` to raise the minimum clock speed makes everything 100% buttery, just like the other less-powerful devices.

```
sudo nvidia-smi --lock-gpu-clocks=1980,3105
```

## 方法三

最正经的解决办法，禁用GSP硬件。注意这里貌似要采用`nvidia`的闭源驱动。

```shell
sudo vim /etc/modprobe.d/nvidia.conf   
```

加上以下选项。

```
options nvidia "NVreg_EnableGpuFirmware=0"
```

然后

```shell
sudo mkinitcpio -P
sudo grub-mkconfig -o /boot/grub/grub.cfg 
```

重启电脑即可。

感受你丝滑的Linux吧!