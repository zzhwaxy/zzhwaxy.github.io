---
layout: post
title: 在Mac Parallels 虚拟机中安装配置Arch Linux
---

## 前言
最近做题目需要Linux环境跑程序。Mac上运行不了，我又不太想用Ubuntu这种发行版。

于是装了个Arch的虚拟机。折腾了一下，差不多勉强能用的程度。

![](/images/arch_neofetch.jpg)

这样看有点花。

做题目的时候其实可以不设置透明度，然后平时用的时候再调回来。

![](/images/arch_pwndbg.jpg)

## 安装系统

首先第一步，检查 ***引导方式*** 。

```
ls /sys/firmware/efi/efivars
```
没有这个文件，说明是BIOS引导，否则是UEFI引导。

### 检查网络

```
ping www.baidu.com -c 4
```
因为在虚拟机里所以默认是正常的。

### 分区

```
cfdisk /dev/sda
```
直接用图形化的方式把硬盘分成一整块，这里选择ISO模式，并且使它可启动。

### 格式化

```
mkfs.ext4 /dev/sda1
```

### 挂载

```
mount /dev/sda1 /mnt
```

### 使用国内镜像

```
vim /etc/pacman.d/mirrorlist
```
用vim把里面China的源移到最上面去。

### 安装系统

```
pacstrap /mnt base base-devel linux linux-headers
```
### 生成分区表

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### 进入新系统

```
arch-chroot /mnt
```

### 安装Bootloader

```
pacman -S os-prober ntfs-3g
```

### 生成引导分区

```
pacman -S grub
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
### 安装必备软件

因为重启后大概率上不了网，所以需要先把网络环境搭建好。

```
pacman -S vim  networkmanager
systemctl enable networkmanager
systemctl start  networkmanager
```

### 退出当前系统

```
exit
```
### 取消挂载

```
umount /mnt
```


### 重启

```
reboot
```

## 安装i3桌面

重启后就可以进行桌面的安装，照我的性格肯定是用i3了。

嫌麻烦随便装一个别的也行。


### 安装i3 

```
pacman -S i3
pacman -S lightdm-gtk-greeter
systemctl enable lightdm
systemctl start lightdm
```

## 配置


### 使用archlinuxcn 源
在 /etc/pacman.conf里加入
```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

```
pacman -Sy archlinuxcn-keyring
pacman -Syy
pacman -S yay
```

### 安装搜狗输入法

```
sudo pacman -S fcitx fcitx-configtool fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5 fcitx-sogoupinyin
```

在.xprofile中添加
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

### 安装浏览器

```
pacman -S firefox
```

### 终端配置

#### SHELL
使用zsh，oh my zsh并配置主题和插件
```
pacman -S zsh 
chsh -s $(which zsh)
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
#### Terminal

```
pacman -S xfce4-terminal
```

#### 终端配色

下载主体并安装

```
git clone https://github.com/netzverweigerer/xfce4-terminal-colorschemes.git
```

#### 字体

```
pacman -S awesome-ttf-font
```
### 文件共享

安装了好几次Parallel tools都不成功。看报错似乎是因为Linux内核太新了不支持，没办法，只能等了。

配个samba先凑合者用吧。

