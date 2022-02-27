 MyArchLinux 安装教程
 
1.再次确保是否为 UEFI 模式

ls /sys/firmware/efi/efivars
2.连接网络

对于有线连接来说，直接插入网线即可。

对于无线连接，则需进行如下操作进行网络连接。

无线连接使用 iwctl 命令进行，按照如下步骤进行网络连接：

iwctl                           #执行iwctl命令，进入交互式命令行

device list                     #列出设备名，比如无线网卡看到叫 wlan0

station wlan0 scan              #扫描网络

station wlan0 get-networks      #列出网络 比如想连接YOUR-WIRELESS-NAME这个无线

station wlan0 connect YOUR-WIRELESS-NAME #进行连接 输入密码即可exit                            #成功后exit退出

接下来正式安装

可以等待几秒等网络建立链接后再进行下面测试网络的操作。

ping www.gnu.org

如果你不能正常连接网络，首先确认系统已经启用网络接口[1]。

ip link  #列出网络接口信息，如不能联网的设备叫wlan0ip link set wlan0 up #比如无线网卡看到叫 wlan0

如果随后看到类似Operation not possible due to RF-kill的报错，

继续尝试rfkill命令来解锁无线网卡。

rfkill unblock wifi

3.更新系统时钟

timedatectl set-ntp true    #将系统时间与网络时间进行同步

timedatectl status          #检查服务状态

4.分区

这里总共设置三个分区，是一个我们认为较为通用的方案。此步骤会清除磁盘中全部内容，请事先确认。

EFI 分区[2]： /efi 800M

根目录： / 100G

用户主目录： /home 剩余全部

这里根目录的大小仅为参考，一般来说个人日常使用的 linux 分配 100G 已经够用了。根目录最小建议不小于 50G，根目录过小会造成无法更新系统软件包等问
题。

通过cfdisk进行操作

5.格式化

建立好分区后，需要对分区用合适的文件系统进行格式化。这里用mkfs.ext4命令格式化根分区与 home 分区，用mkfs.vfat命令格式化 EFI 分区。如下命令中
的 sdax 中，x 代表分区的序号。格式化命令要与上一步分区中生成的分区名字对应才可以。如果你还是不会操作，可参见配套视频中的详细操作。

磁盘若事先有数据，会提示你: 'proceed any way?' 按 y 回车继续即可。

mkfs.fat -F32 /dev/nvme0n1p1 

mkswap /dev/nvme0n1p2

swapon /dev/nvme0n1p2 

mkfs.ext4  /dev/nvme0n1p3

mkfs.ext4  /dev/nvme0n1p4

6.挂载

在挂载时，挂载是有顺序的，先挂载根分区，再挂载 EFI 分区。 这里的 sdax 只是例子，具体根据你的实际情况来，如果你还是无法理解，请注意配套视频中的
操作。

mount /dev/nvme0n1p3  /mnt #切记一定要先挂载根目录

mkdir /mnt/boot    #创建efi目录

mount /dev/nvme0n1p1  /mnt/boot

mkdir /mnt/home    #创建home目录

mount /dev/nvme0n1p4 /mnt/home

设置pacman服务器地址

cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.backup

reflector -c China -a 5 --sort rate --save /etc/pacman.d/mirrorlist 

8.安装系统

必须的基础包

pacstrap /mnt base base-devel linux linux-headers linux-firmware  #base-devel在AUR包的安装是必须的

必须的功能性软件

pacstrap /mnt dhcpcd iwd vim bash-completion   #一个有线所需(iwd也需要dhcpcd) 一个无线所需 一个编辑器 一个补全工具

9.生成 fstab 文件


fstab 用来定义磁盘分区

genfstab -U /mnt >> /mnt/etc/fstab

复查一下 /mnt/etc/fstab 确保没有错误

10.change root

把环境切换到新系统的/mnt 下

arch-chroot /mnt

11.时区设置

设置时区，在/etc/localtime 下用/usr 中合适的时区创建符号连接。如下设置上海时区。

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

随后进行硬件时间设置，将当前的正确 UTC 时间写入硬件时间。

hwclock --systohc

12.设置 Locale 进行本地化

Locale 决定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

编辑 /etc/locale.gen，去掉 en_US.UTF-8 所在行以及 zh_CN.UTF-8 所在行的注释符号（#）。

然后使用如下命令生成 locale。

locale-gen

向 /etc/locale.conf 导入内容

echo 'LANG=en_US.UTF-8'  > /etc/locale.conf

13.设置主机名

首先在/etc/hostname设置主机名

vim /etc/hostname

加入你想为主机取的主机名，这里比如叫 KB。

接下来在/etc/hosts设置与其匹配的条目。

vim /etc/hosts

加入如下内容

127.0.0.1   localhost

::1         localhost

127.0.1.1   KB.localdomain KB

14.为 root 用户设置密码

passwd root

15.安装微码

pacman -S intel-ucode   #Intel

pacman -S amd-ucode     #AMD

16.创建用户，安装引导

useradd -m -g users -G wheel,storage,power -s /bin/bash kongbai

passwd kongbai

EDITOR=vim visudo 

找到下面这样的一行，把前面的注释符号 # 去掉，在在最后一行加入

Defaults rootpw :wq 保存并退出即可。

#%wheel ALL=(ALL) ALL

bootctl install 

vim  /boot/loader/entries/arch.conf

在里边加入

title Arch Linux

linux /vmlinuz-linux

initrd /intel-ucode.img

initrd /initramfs-linux.img

echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p3) rw" >> /boot/loader/entries/arch.conf

开启32位软件库和archlinuxcn源

vim /etc/pacman.conf

去掉[multilib]一节中两行的注释，来开启 32 位库支持。

在最后加入

[archlinuxcn]

Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

最后:wq 保存退出，刷新 pacman 数据库

pacman -Syyu

17.完成安装

exit                # 退回安装环境#

umount -R  /mnt     # 卸载新分区

reboot              # 重启

注意，重启前要先拔掉优盘，否则你重启后还是进安装程序而不是安装好的系统。重启后，开启 dhcp 服务，即可连接网络

systemctl enable dhcpcd  #立即启动dhcpping www.gnu.org      #测试网络连接

若为无线链接，则还需要启动 iwd 才可以使用 iwctl 连接网络

systemctl enable iwd #立即启动iwd

iwctl               #和之前的方式一样，连接无线网络

sudo pacman -S xf86-video-amdgpu 显卡驱动

音频、蓝牙

驱动安装

sudo pacman -S alsa-utils  pulseaudio-alsa soft-firmware  音频

sudo pacman -S  bluez bluez-utils pulseaudio-bluetooth pavucontrol  蓝牙

sudo pacman -S  archlinuxcn-keyring 

sudo pacman -S  yay

sudo pacman -S  git

sudo pacman -S  acpi neofetch

使用pacman 安装sddm，并设置服务开机自启 

sudo pacman -S sddm

sudo systemctl enable sddm

12 

接着创建启动项 

新建文件 /usr/share/xsessions/dwm.desktop, 中间如果某个目录没有，则创建它 在dwm.desktop 中添加如下内容 

[Desktop Entry]

Encoding=UTF-8

Name=Dwm

Comment=Dynamic window manager

Exec=dwm

Icon=dwm

Type=XSession

重启之后就可以进入登陆界面了，输入用户和密码就可以进入系统，这个时候也可以看到直接就进入到dwm窗口了

sudo pacman -S xorg-server xorg-apps xorg-xinit chromium pcmanfm
