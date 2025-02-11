---
title: Arch安装记录
date: 2025-02-11 17:03:59 +08:00
filename: 2025-02-11-Arch-install
categories:
  - Misc
tags:
  - Environment
  - Arch
  - Linux
dir: Misc
share: true
---
## 环境和准备工作

环境：联想拯救者R9000P 2021 Ryzen R7 5800H RTX 3060

跟着这个[教程](https://www.bilibili.com/video/BV1CCmiYKETP/?spm_id_from=333.880.my_history.page.click&vd_source=a050735bf251f44101103e1314e38fe9)走的

先在Windows上分区，这样可以节省后面在Arch安装过程中所耗费的时间。

格式感觉无所谓，反正待会安装过程中还是要格式化一遍磁盘的。

**！！！注意**

我采用Linux的boot分区和Windows独立的分区方案，如果你想要Windows和Linux的boot在一个分区的话，请务必清楚你在干什么，由于本篇文章仅作为个人记录，所以不对另一种方案做详细讲解。

我打算分给Arch 500G，分区方案如下

`/boot` 1G
`swap` 16G
`/` 483G

实际上swap没必要分这么大，但是我盘的空间够，就直接分了。

各位在分盘的时候保证给`/boot`和`/swap`的空间，剩下的可以全给根目录。

## 下载镜像，烧录

烧录工具可以用[balenaetcher](https://etcher.balena.io/)

如果你有[Ventoy](https://www.ventoy.net/cn/)U盘，也可以直接把下载下来的镜像丢到ISO目录。

## Arch安装

在电脑启动的时候将启动项改为你刚刚准备的镜像U盘。选中Arch，然后默认直接按回车就好。

等待一会就会进入到安装页面，此时输入

```shell
iwctl
```

就会进入到联网页面

```shell
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡看到叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 wifi 网络
station wlan0 connect wifi-name # 进行连接，注意这里无法输入中文。回车后输入密码即可
exit # 连接成功后退出
```

你可能发现你的`wlan0`扫不出来网，别急，先试试输入
```shell
rfkill unblock all
```

然后再试试。据说有些网卡还不兼容，只能用网线的方式。听说用手机共享热点然后用数据线也可以。

### 磁盘分区

然后我们就开始着手磁盘分区。

输入
```
lsblk
```

即可看到当前分区情况，记下你刚刚分的区的名字。
比如我的大概就是如下这样。
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    1     0B  0 disk 
sdb           8:16   1     0B  0 disk 
nvme1n1     259:0    0   1.8T  0 disk 
├─nvme1n1p1 259:1    0   128M  0 part 
├─nvme1n1p2 259:2    0 962.9G  0 part 
├─nvme1n1p3 259:3    0   272G  0 part 
├─nvme1n1p4 259:4    0   128G  0 part 
├─nvme1n1p5 259:5    0     1G  0 part 
├─nvme1n1p6 259:6    0    16G  0 part 
└─nvme1n1p7 259:7    0   483G  0 part 
nvme0n1     259:8    0 476.9G  0 disk 
├─nvme0n1p1 259:9    0   260M  0 part 
├─nvme0n1p2 259:10   0    16M  0 part 
├─nvme0n1p3 259:11   0   200G  0 part 
├─nvme0n1p4 259:12   0 274.7G  0 part 
└─nvme0n1p5 259:13   0     2G  0 part 
```

可以看见`nvme1n1p5` `nvme1n1p6` `nvme1n1p7`这三个分区就是我所需要的分区。

然后我们进行格式化分区的操作。根目录格式化成`ext4`，boot分区格式化成`fat32`，swap分区格式化为`linuxswap`，命令如下

```shell
mkfs.vfat -F32 /dev/nvme1n1p5
mkswap /dev/nvme1n1p4
mkfs.ext4 /dev/nvme1n1p7
```

接着我们来挂载分区。

```shell
mount /dev/nvme1n1p7 /mnt # 这里是挂载根目录
```

接着挂载boot分区。我采用Linux和Windows独立boot分区，所以我先根目录创建boot目录。

```shell
mkdir -p /mnt/boot
mount /dev/nvme1n1p5 /mnt/boot
```

如果你想要Windows和boot挂一个分区的话，你就需要先确认Windows 的boot所在位置，挂载步骤是一样的，如果这样的话在之前分盘的时候就不需要分boot分区了，可以把Windows 的boot分区扩大一点。

swap分区无需挂载，但需要加入以下命令启用

```shell
swapon /dev/nvme1n1p6
```

最后输入
```shell
lsblk
```
就能看见分区情况了，大概如下图所示。

```
├─nvme1n1p5 259:5    0     1G  0 part /boot
├─nvme1n1p6 259:6    0    16G  0 part [SWAP]
└─nvme1n1p7 259:7    0   483G  0 part / 
```

### 换源，安装基本软件

输入以下命令自动替换国内镜像源
```shell
reflector --country 'China' --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist --download-timeout 60
```

这里的`--download-timeout 60`是用来指定超时时间的，如果网不好的情况下请加上这个参数，网好的话不用加。

打开`multilib`库，将以下字段前面的`#`删除
```shell
[multilib]
Include = /etc/pacman.d/mirrorlist
```

刷新镜像源
```
pacman -Syy
```

安装基本系统

```shell
pacstrap /mnt base base-devel linux-zen linux-zen-headers linux-firmware vim nano e2fsprogs ntfs-3g
```

这里我没有安装默认的`Linux`内核，而是直接选择`Linux-zen`内核，如果你想知道内核之间的具体区别，请查阅Arch Wiki。

更新密钥

```shell
pacman -Sy achlinux-keyring
```

这个不知道是干嘛的，但是跟着敲就完了。

输入以下命令生成`fstab`文件

```shell
genfstab -U -p /mnt >> /mnt/etc/fstab
```

完成之后输入
```shell
vim /mnt/etc/fstab
```
里面有一行行的内容就是成功了。

输入以下命令进入新系统

```shell
arch-chroot /mnt
```

注：（现在就是进入了新系统的根目录了（/mnt），所有的操作都会保留在新的arch系统里了。相当于进入了没有界面的系统。后续系统坏了挂载磁盘后也是用这个命令进入系统修复即可。

输入以下命令来设置`root`用户密码。

```shell
passwd
```

执行以下命令新建一个隶属于 wheel 组的新用户（之前创建的是root用户，正常不会用root用户使用系统。现在建一个普通用户，用户名随意取个，我取得是yu
```shell
useradd -m -G wheel -s /bin/bash 你的用户名
passwd 你的用户名 # 给你的用户设置密码
```

为了使普通用户能获取临时的`root`权限，我们需要修改`wheel`的权限设置
```shell
vim /etc/sudoers
```

找到以下内容
```
## Uncomment to allow members of group wheel to execute any command 
# %wheel ALL=（ALL）ALL
```

删除第二行前面的#，然后变成：

```
## Uncomment to allow members of group wheel to execute any command 
%wheel ALL=（ALL）ALL
```
保存退出。

安装微码，根据自己的CPU选择其中一条命令。
```shell
pacman -S intel-ucode
pacman -S amd-ucode
```

根据提示按`y`安装

安装`Grub`引导，也就是双系统选择系统的入口。
```shell
pacman -S grub efibootmgr os-prober
```

安装`Grub`配置文件
```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --recheck
```

修改`Grub`配置文件
```shell
vim /etc/default/grub
```

找到
```
#GRUB_DISABLE_OS_PROBER=false
```
这一行，删除前面的`#`，也就是取消注释，这样才能找到`Windows`的入口。

生成`Grub`配置文件。
```shell
grub-mkconfig -o /boot/grub/grub.cfg
```
此时可能还找不到`Windows`引导，没关系，待会进系统再输一次就能找到了。

配置语言和区域
```shell
vim /etc/locale.gen
```

分别找到以下字符，删除最前面的`#`
```
#en_US.UTF-8 UTF-8
#zh_CN.UTF-8 UTF-8
#zh_TW.UTF-8 UTF-8
```

然后输入
```shell
locale-gen
```
刷新区域信息

```shell
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

输入如下命令设置时区为上海
```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

安装`kde`
```shell
pacman -S plasma sddm konsole dolphin ark gwenview
```

设置`SDDM`开机自启
```shell
systemctl enable sddm
```

安装字体
```shell
pacman -S wqy-microhei
```

安装`NetworkManager`，用于管理网络连接
```shell
pacman -S networkmanager
systemctl enable NetworkManager
```

安装输入法和浏览器
```shell
pacman -S firefox
pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-rime
pacman -S wget
```

输入法需要设置环境变量
```shell
vim /etc/environment
```

加入以下几行
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

安装`Nvidia`闭源驱动，如果你和我一样是3060且使用`Linux-zen`内核的话，可以直接输入我的命令，如果是其他内核和不同的显卡型号，请参考Arch Wiki
```shell
pacman -S nvidia-dkms nvidia-utils nvidia-settings lib32-opencl-nvidia lib32-nvidia-utils
```

其实安装已经完成了，现在你就可以重启了。

```
exit
reboot
```

即可来到新系统。重启的时候屏幕亮了记得拔下U盘。然后就是进入系统了别忘记
```shell
sudo grub-config -o /boot/grub/grub.cfg
```

剩下的就是额外的操作了

## Paru

1. 安装 `paru`。
    
在 `/etc/pacman.conf` 文件中写入下列内容。
    
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch
Server = https://repo.huaweicloud.com/archlinuxcn/$arch
```

```
sudo pacman-key --lsign-key "farseerfc@archlinux.org"
sudo pacman -S archlinuxcn-keyring
sudo pacman -Sy
sudo pacman -S paru
```

2. 安装 `clash-verge-rev-bin`。
    
```
paru clash-verge-rev-bin
```

注意，这里选择`archlinuxcn`的源，这样就不用`curl`了，现阶段网络环境大概率`curl`不下来。


## 系统软件

### 蓝牙

默认情况想要使用蓝牙还得安装以下两个包，然后再启用蓝牙服务。

```shell
paru -S bluez bluez-utils
systemctl enable bluetooth.service
systemctl start bluetooth.service
```

### 打印机服务

点击kde设置里面的打印机，能看见提示，需要先安装一个组件，我忘记叫什么了，如果有人知道，可以在评论区告诉我一下，我会加上。

```shell
paru -S cups
systemctl enable cups
```

### 电源方案显示
```shell
paru -S power-profiles-daemon 
```

## 美化

### SDDM

采用[这个主题](https://github.com/Keyitdev/sddm-astronaut-theme)。

可以输入以下命令快速安装。

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/keyitdev/sddm-astronaut-theme/master/setup.sh)"
```

## Grub

采用这个[主题](https://github.com/shvchk/fallout-grub-theme)

可以采取以下方式快速安装。

```
wget -O- https://github.com/shvchk/fallout-grub-theme/raw/master/install.sh | bash -s -- --lang Chinese_simplified
```

## Nvidia

我采用独显直连，但是发现在Wayland下Nvidia会有奇怪的Bug，体现在频繁掉帧，输入法快速打字响应很慢等问题，所以最后还是选择用X11。

> So Nvidia,fuck you!

### 关于Nvidia驱动

我用的是Linux-zen内核，所以选择闭源的`nvidia-dkms`驱动。

可以直接安装一下的包（仓库源要打开multilib，不然装不了32位库）

```shell
paru -S nvidia-dkms nvidia-utils nvidia-settings lib32-opencl-nvidia lib32-nvidia-utils
```

为了能正常玩Steam，我又下了如下两个包

```shell
paru -S lib32-vulkan-icd-loader vulkan-icd-loader
```

为了充分释放显卡性能，我们还需要启用以下服务，不然卡会锁功耗。

```shell
systemctl enable nvidia-powerd.service
systemctl start nvidia-powerd.service
```

## 后面的就是本人的私人配置

仅做记录，大家可以忽略

## Kitty终端

```shell
paru -S kitty
```

```shell
vim ~/.config/kitty/kitty.conf
```

```
#font
font_size 12.0
font_family Hack Nerd Font Mono
bold_font auto
italic_font auto
bold_italic_font auto

#window
background_opacity 0.8
background_blur 64
remember_window_size yes

#tab bar
tab_bar_edge top
tab_bar_style powerline
tab_powerline_style round

cursor_trail 3

#BEGIN_KITTY_THEME
#Cobalt Neon
include current-theme.conf

#END_KITTY_THEME
```

## 没错，还是zsh及其美化

```shell
paru -S zsh
chsh -s /usr/bin/zsh # 修改默认终端为zsh，注意这个更改要注销再登录才能发生
```

注销并再次登录,打开shell  
这时会看到

```
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

--- Type one of the keys in parentheses --- 
```

输入0就会退出并生成配置文件~/.zshrc

下一步是装`oh-my-zsh`，我特地选的`gitee`仓库，国内能很方便的装。
```
wget https://gitee.com/pocmon/ohmyzsh/raw/master/tools/install.sh  

chmod +x install.sh 

./install.sh
```

#### 安装p10k主题

首先要装上依赖字体

[安装文档](https://github.com/ryanoasis/nerd-fonts#font-installation)

不过我们有`paru`，可以很方便的装

```shell
paru -S ttf-hack-nerd
```

接下来我们安装`p10k`

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

在`~/.zshrc`中修改

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

执行`source ~/.zshrc`即可进入p10k配置，根据提示完成配置

#### zsh插件配置

下面是`zsh`插件

```bash
// 克隆两个插件
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```bash
// 配置插件
plugins=(git zsh-syntax-highlighting zsh-autosuggestions z)
```

至此，我们的美化工作已经结束。


## Docker镜像包导入

安装`Docker`
```shell
paru -S docker
sudo systemctl start docker
sudo systemctl enable docker
```

恢复之前的容器
```shell
sudo docker load -i 15445-image.tar
sudo docker run -d --name Ubuntu-22 --hostname Legion-Docker -p 2222:22 -p 5174:5173 15445-image # 5173是vue端口
sudo docker update --restart unless-stopped Ubuntu-22
```

## Frp配置

```shell
sudo vim /etc/systemd/system/frpc.service
```

```
[Unit]
# 服务名称，可自定义
Description = frpc server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart =/home/yu/frp/frpc -c /home/yu/frp/frpc.toml
Restart=always
RestartSec=30
User=root
Group=root

[Install]
WantedBy = multi-user.target
```

```shell
sudo systemctl enable frpc
sudo systemctl start frpc
```