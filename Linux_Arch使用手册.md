# Arch Linux使用手册

## 概述  

### 编写目的  

#### 目的阐述  

+  为项目的开发设计的依据与指导。  
+  为参与该项目的编程人员提供依据；  
+  为修改、维护提供条件；  
+  项目负责人将按计划书的要求布置和控制开发工作全过程；  
+  项目质量保证组将按此计划书做阶段性和总结性的质量验证和确认。  

#### 读者对象  

+  项目开发人员，特别是编程人员；  
+  软件维护人员；  
+  技术管理人员；  
+  执行软件质量保证计划的专门人员；  
+  参与本项目开发进程各阶段验证、确认以及负责为最后项目验收、鉴定提供相应报告的有关人员。  
+  合作各方有关部门的负责人；项目组负责人和全体参加人员。  

#### 注意事项  

+  权限审查：此文档仅供技术部功能组内部使用。  
+  保存备份：此文档允许修改，允许本地备份。  
+  该文档采用 markdown 编写规范，建议使用markdownPad查看和修改。  

## 安装

---
```
    1、制作U盘启动器
        下载IOS镜像
        ~~安装UltralOS或者PowerIOS~~
        windows下用USBWrite刷入磁盘映像
        刷入映像

    2、进入电脑Bios选择设置为U盘启动
        a、重启U盘启动 按F8 等进入菜单选择
       
        b、查看引导方式 是BIOS 或是EFI方式引导
            fdisk -l
            ls /sys/firmware/efi/efivars

        3、连接网络
            方式一：有线
                dhcpcd      // 自动分配IP
                  

            方式二：无线连接
                ip addr                          // 查看网络硬件设备
                ip link set wlan0 up             // 打开无线设备wlan0
                iwlist wlan0 scan                // 用无线设备wlan0扫描查看附近wifi列表
                iwlist wlan0 scan | grep ESSID   // 用无线设备wlan0扫描查看附近wifi列表并过滤只显示wifi名
                
                wpa_passphrase 网络 密码 > 文件名 // 将网络和密码信息保存到一个配置文件中
                例： wpa_passphrase Xiaomi_26DE xzh250xxxx > internet.conf


                wpa_supplicant -c 文件名 -i 设备名 & // 后台挂起执行，用配置文
                件的内容使用wifi设备连接
                例子： wap_supplicant -c internet.conf -i wlan0 &
                注： 如果一直在刷日志可能是密码不对，重新生成网络信息文件再次连
                接（用 ps 查看进程 kill -9 进程号 杀死进程）


                dhcpcd  // 开启自动获取IP地址

                ping baidu.com   // ping 百度网址查看是否连接成功
        
        4、同步时间
            timedatectl set-ntp true   // 同步校对时间
            date                       // 查看日期与时间

        5、分区
            fdisk -l                  // 查看硬盘列表
            lsblk
			
			格式化 
				法一：fdisk
					fdisk 位置                // 进入磁盘位置设置
					例： fdisk /dev/sdb
					
				法二：cfdisk
				
				法三：parted
					(parted) select /dev/sda
					(parted) mklabel gpt
					(parted) mkpart primary 1 512M
					(parted) mkpart primary 512M 25G
					(parted) mkpart primary 25G -1
					(parted) set 1 boot on
					(parted) p
					(parted) q
					
					第一行选择操作硬盘为 /dev/sda，或启动时选择硬盘：parted /dev/sda
					第二行建立 gpt 分区表，或简历mbr分区表：mklabel msdos
					第三行建立的分区用作 ESP
					第四行建立的分区用作 LVM，(不需要设定分区标记为lvm)
					第五行建立剩余的分区备用，结束参数为-1，即从尾部计数
					第六行设定 ESP 分区标志：boot
					第七行打印分区信息，p(rint)
					第八行退出parted，q(uit)
					
					注：parted命令更改磁盘分区格式按照习惯MBR格式一般在linux下称作dos msdos
					
			格式化分区，并挂载（efi）
				mkfs.ext4 /dev/sda3            #以ext4方式格式化磁盘/dev/sda的/dev/sda3分区
				mkfs.vfat -F32 /dev/sda1       #以vfat方式创建efi
				mount /dev/sda3 /mnt           #挂载/
				mkdir -p /mnt/boot/efi         #建立boot文件夹
				mount /dev/sda1 /mnt/boot/efi  #挂载efi
				
			格式化分区，并挂载（mbr）
				mkfs.ext4 /dev/sda3            #以ext4方式格式化磁盘/dev/sda的/dev/sda3分区
				mkfs.ext4 /dev/sda1
				mount /dev/sda3 /mnt           #挂载/
				mkdir -p /mnt/boot             #建立boot文件夹
				mount /dev/sda1 /mnt/boot      #挂载/boot
				
			创建并启用swap分区
				mkswap /dev/sda2  #创建swap分区
				swapon /dev/sda2  #启用swap分区

        6、设置镜像源地址顺序
				vim /ect/pacman.conf     // 打开镜像源配置位置 调整顺序
				将color 行前的# 去掉
				vim /ect/pacman.d/mirrorlist  // 调整软件源选用顺序
				
            或使用网易的镜像源
				grep 163 /etc/pacman.d/mirrorlist > bak
				cat bak > /etc/pacman.d/mirrorlist
				pacman -Syy
				
			
			安装基本包
				pacstrap /mnt base base-devel   linux                      // 安装基本操作系统
				或
				pacstrap /mnt base base-devel linux linux-firmware    
				
			生成fstab
				genfstab -U /mnt >> /mnt/etc/fstab // 将分区挂载信息写入到fstab中

			查看fstab
				cat /mnt/etc/fstab
  

        7、进入系统
            arch-chroot /mnt

			安装编辑器vim
				pacman -S vim
			配置时间
				ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime   // 配置时间
				hwclock  --systohc    // 同步时间

            配置语言
				vim /etc/locale.gen     // 去掉打开的文件的中的 en.US UTF-8所在行的注释符
				locale-gen
				vim /etc/locale.conf    // 在文件中 增LANG=en_US.UTF-8
			
			设置主机名			
				echo CodeRooster > /etc/hostname  // 设置主机名
				
            创建密码
				passwd   // 创建密码
				
            安装wifi工具包
				pacman -S wireless_tools wpa_supplicant dhcpcd   // 网络连接套餐
				pacman -S broadcom-wl-dkms （wl模块 modprobe wl）

			安装系统引导(pacman -S dosfstools grub efibootmgr os-prober)

				安装引导程序 grub 和 efi管理工具
					pacman -S grub efibootmgr --noconfirm

				如安装有多系统 需安装 os-prober
					pacman -S os-prober


				安装引导工具
					pacman -S dosfstools grub efibootmgr os-prober

				安装引导  使用了efi的情况
					grub-install --efi-directory=/boot --bootloader-id=grub

			配置grub
				生成引导配置
					grub-mkconfig -o /boot/grub/grub.cfg

				其他
					pacman -S grub efibootmgr intel-ucode amd-ucode os-prober dosfstools ntfs-3g // 启动器管理套餐 安装引导程序（UEFI）
					uname -m   // 确认系统架构
					
					// gpt引导的grub设置（注: 需要制作的启动盘是UEFI启动进来安装才不会报错，不然无法成功）
					mkdir /boot/efi
					grub-install --target=x86_64-efi --efi-directory=boot
					grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchLinuxGrub
					
                    // magr引导的grub设置
					grub-install /dev/sdb    // magr引导的grub设置
				
					grub-mkconfig -o  /boot/grub/grub.cfg


           退出chroot重启
				exit
				umount -R /mnt
				reboot 

            其他
                pacman -S networkmanager net-tools //安装NetWorkManager
                
                vim /etc/hosts 添加下面条目
                127.0.0.1 localhost
                ::1 localhost

```

+ 问题记录

```

    U盘启动后选择Arch界面菜单后进入不到
    报错信息:
    ...
    Waiting 30 seconds for device /dev/disk/by-label/ARCH_202002 ...
    ERROR:'/dev/disk/by-label/ARCH_202002' device did not show up after 30
    seconds...
        Falling back to interactive prompt
        You can try to fix the problem manually, log out when you are finished
    sh: can't access tty; job cantrol turned off
    可能问题：
    镜像写入有问题：用USBWrite重新写入。


    failed to  get canonical path of airootfs

    没切换 arch-chroot /mnt
```


```
	grub-install:error: efibootmgr failed to register the boot entry:no such file or directory

	问题可能是efivars内核模块没有加载

	然后用modprobe试了一下，果然如此

	#modprobe efivars

	error: FATAL:module efivars not found in directory /lib/modules/5.2.5-arch/-1.ARCH

	然后思路就清晰了，把这个模块装上去就可以了

	第一种方法：#mount --bind /sys/firmware/efi/efivars /mnt/sys/firmware/efi/efivars

	第二种方法：#sudo pacman -S efibootmgr

	然后#grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ArchlinuxGrub --recheck

	#grub-mkconfig -o /boot/grub/grub.cfg

```

> 参考资料
- [archlinux安裝手记（Win10+Arch、GPT+UEFI、lvm）](https://www.cnblogs.com/unkownarea/articles/6472461.html)

### windows和linux双系统

```
	安装ntfs-3g，再mkconfig
	
		pacman -S ntfs-3g
		grub-mkconfig -o /boot/grub/grub.cfg  
	
	
	
	gpt的硬盘，安装双系统，进入bios设置为只用UEFI；
	
	先安装windows10，使用为indows的PE启动盘，进行格式化；安装windows10时会多出来两个盘，不要删除。之后archlinux的grub引导也安装在esp标识的盘上面
	
	安装ArchLinux时需用UEFI引导，进入bios设置为只用UEFI，启动U盘时用 rufus制作，在引导类型上选上UEFI
	进行grub引导安装时，记得挂在windows安装时的那个盘，挂到archlinux的/boot/efi下 再执行 grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --rechech
	进入启动盘时用UEFI的启动方式进入安装，不让用grub设置引导驱动时会报错无法进入下一步。
	
	
```

```
首先，按正常方式安装Arch一直到grub的地方；
然后：一、如果用grub2引导1.如果是MBR:
grub-install /dev/sdX
(X是主引导盘，主板优先启动的物理硬盘，其实应该哪一块硬盘都无所谓，可以bios里改）
然后 
grub-mkconfig -o /boot/grub/grub.cfg
注意这之前要在arch里安装好os-prober，不然grub识别不到Windows。

如果安装过程中os-prober出问题，可以先卸掉，进系统后装上，重新grub-mkconfig，Windows就有了

2.如果是GPT+UEFI:

grub-install --efi-directory=/boot/EFI

grub-mkconfig -o /boot/grub/grub.cfg

后面/boot/EFI要改成自己的EFI目录，我个人是把ESP挂载到/boot/EFI的然后重复上文grub-mkconfig部分 完工

```

> 参考资料

+ [ArchLinux(简体中文)](https://wiki.archlinux.org/index.php/Main_page_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
+ [WIN10 + ArchLinux双系统安装](https://www.jianshu.com/p/ef24cca7a2d1)
+ [以官方Wiki的方式安装ArchLinux](https://www.viseator.com/2017/05/17/arch_install/)
+ [ArchLinux安装](https://www.jianshu.com/p/03447eee8953)
+ [在双硬盘下，第一个硬盘安装好win10后，第二个全新硬盘，如果安装archlinux加引导?](https://www.zhihu.com/question/369863843/answer/1001971197)

### 添加用户

---
```

    useradd -m -G wheel coderooster  // 添加一个名为coderooster的用户其会生成家目录并加入wheel组
    
    passwd coderooster   // 给coderooster更改密码

    给wheel提权

    ln -s /usr/bin/vim /usr/bin/vi    // vi没安装，把vim映射到vi
    visudo   // 进入配置文件 去掉 wheel 前的注释

    exit     // 登出  换用户

	这里说下你可以sudoers添加下面四行中任意一条
	youuser            ALL=(ALL)                ALL
	%youuser           ALL=(ALL)                ALL
	youuser            ALL=(ALL)                NOPASSWD: ALL
	%youuser           ALL=(ALL)                NOPASSWD: ALL

	第一行:允许用户youuser执行sudo命令(需要输入密码).
	第二行:允许用户组youuser里面的用户执行sudo命令(需要输入密码).
	第三行:允许用户youuser执行sudo命令,并且在执行的时候不输入密码.
	第四行:允许用户组youuser里面的用户执行sudo命令,并且在执行的时候不输入密码.

	4.撤销sudoers文件写权限,命令:
	chmod u-w /etc/sudoers

```

> 参考资料

+ [ArchLinux安装后的必须配置与图形界面安装教程](https://www.viseator.com/2017/05/19/arch_setup/)
+ [archlinux 中文本地化配置](https://blog.csdn.net/holdsky/article/details/8515143)
+ [Localization/Simplified Chinese (简体中文)](https://wiki.archlinux.org/index.php/Localization/Simplified_Chinese_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
+ [Arch Linux中文乱码解决](https://www.cnblogs.com/bluestorm/p/6039197.html)
+ [xxx is not in the sudoers file.This incident will be reported.的解决方法](https://www.cnblogs.com/zox2011/archive/2013/05/28/3103824.html)

### 时间同步

```
sudo timedatectl set-ntp true


时钟软件
sudo pacman -S xorg-xclock

使用 man xclock

例子：
xclock -d -update 1 -strftime % T %

```

+ 方法二
```
sudo pacman -S ntp

yay -S ntpdate

sudo ntpdate cn.pool.ntp.org
```


### 查看用户和用户组

+ 查看当前活动的用户
```
	命令： w
```

+ 查看该linux中所有的用户
```
	命令：cat /etc/passwd
```

+ 查看当前登录用户归属的用户组
```
	命令： groups
```
+ 查看该linux中的所有用户组
```
	命令: cat /etc/group
```
+ 新建工作组
```
	命令: groupadd [groupname]
```
+ 将用户添加进工作组
```
	命令: usermod -G [groupname] [username]
```
+ 将已存在的用户加入到用户组中
```
	命令: sudo usermod -a -G audio coderooster

	注： 将用户coderooster 添加到audio组中
```

+ 将用户从用户组中删除
```
	命令： gpasswd -d coderooster audio
```
+ 用户账户
```
	超级用户root（0）
	程序用户（1~499）
	普通用户（500~65535）
```

> 参考资料
- [linux中如何查看用户和用户组_电脑软件-百度经验](https://jingyan.baidu.com/album/e75aca8583eec8542fdac617.html?picindex=3)

- [Linux系统的用户和用户组管理 - 钟桂耀 - 博客园](https://www.cnblogs.com/zhongguiyao/p/9165917.html)

### 中文环境配置

```
    中文本地化配置
    vim /etc/locale.gen     // 去掉打开的文件的中的 en.US UTF-8所在行的注释符
        zh_CN.GB18030  GB18030
        zh_CN.UTF-8  UTF8
   
    sudo pacman -S wqy-zenhei // 安装字体

    locale-gen   // 执行命令，生成语言包

    vim /etc/locale.conf    // 在文件中 增LANG=en_US.UTF-8
```

+ 安装字体
```
sudo pacman -S ttf-dejavu 
yay -S ttf-symbola unicode-emoji

sudo pacman -S wqy-micorhei    #文泉驿微米黑
sudo pacman -S adobe-source-han-sans-cn-fonts    # 思源黑体
sudo pacman -S ttf-arphic-uming    #文鼎明体
```
- 终端等宽字体，如ttf-dejavu
- 数学和符号字体，如ttf-symbola（包含emoji表情，emoji也可安装noto-fonts-emoji ）

> 参考资料
- [archlinux安裝手记（Win10+Arch、GPT+UEFI、lvm）](https://www.cnblogs.com/unkownarea/articles/6472461.html)

### 更新方法

```
pacman -Syu    #升级整个系统
pacman -S name    #安装软件
pacman -Ss words    #查询有某关键字的软件 words即是要查询的关键字
pacman -R name    #移除某软件但不移除其依赖
pacman -Qi name   #查看已经安装的某软件的信息
```

```
    
    pacman -Syyu    // 更新


    pacman -S man   // 手册查看
    
    pacman -S base-devel


	为Arch Linux更换Archlinuxcn源（清华源）

	具体操作方法：
	# sudo vim /etc/pacman.conf
	文件末尾添加以下两行：
	[archlinuxcn]
	Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

	然后很多教程在这里配置就结束了，就遇到一些问题，比如我们安装某些软件的时候会提示gpg签名错误损坏等这些，这个是因为没有导入key的原因，我们需要导入一下GPG KEY，具体操作就是
	安装archlinux-keyring 包导入 GPG key。
	# sudo pacman -S archlinux-keyring
	再更新一下源：
	# sudo pacman -Sy
```

## 驱动

### 声卡

```
    安装：
        pacman -S xf86-video-vesa # 通用显卡驱动，不提供任何2D和3D加速功能
         
        pacman -S xf86-video-intel # Intel
        pacman -S xf86-video-nouveau # Nvidia
        pacman -S nouveau-dri
        pacman -S xf86-video-ati # Ati
         
        pcaman -S xf86-video-vesa \\安装声卡驱动
        pacman -S alsa-utils
         
    使用：
        调节音量
        1，使用前确认安装了alsa-utils
    
            sudo pacman -S alsa-utils
        2，运行alsamixer调试音量
            alsamixer
            左右键选择调哪个，将Master和PCM按“m”解除静音（下边MM变成00），使用上下键调大音量（本人两个都调到了79）基本放歌就能听到声音了
    
        3，Esc退出，保存设置，运行
            sudo alsactl store
```

```
	没有声音:
	排查:   1. alsamixer 查看MM 的设备,按 m 关闭静音模式
			2. 切换对播放设备
			3. 查看设备权限 ls -la /dev/snd  来查看音频设备的使用权限。

```

```
安装 pulseaudio 
有的 PulseAudio 模块已经从主软件包中 分离 了。如果需要的话请分别安装：

pulseaudio-bluetooth: 蓝牙 (Bluez) 支持
pulseaudio-equalizer: 均衡器 接收器 (qpaeq)
pulseaudio-gconf[断开的链接：package not found]: GConf 支持 (paprefs)
pulseaudio-jack: JACK 接收器, 源和jackdbus检测
pulseaudio-lirc: 红外 (LIRC) 音量控制
pulseaudio-xen[断开的链接：package not found]: Xen 半虚拟化接收器
pulseaudio-zeroconf: Zeroconf (Avahi/DNS-SD) 支持
```

> Note: 用户可能在 ALSA 和 PulseAudio 之间产生混淆。ALSA 包括有着声卡驱动的Linux内核组件和用户空间组件两部分，libalsa.PulseAudio 只依赖于内核组件,但是也通过 pulseaudio-alsa 和 libalsa 实现了兼容。

+ 前端
> 有多种前端工具可以用以控制 PulseAudio 守护进程：
	- GTK GUI：paprefs 和 pavucontrol
	- 按键音量控制: pulseaudio-ctlAUR，pavolume-gitAUR
	- 控制台 (CLI)混合器：ponymix和pamixer
	- 控制台 (curses) 混合器：pulsemixer
	- 网页音量控制：PaWebControl
	- 系统托盘图标：pasystray，pasystray-gitAUR
	- KF5 plasma 程序：kmix和plasma-pa
	> 如果你想通过 PulseAudio 使用蓝牙耳机或其他蓝牙音频设备的话，请参见Bluetooth headset.

```
	sudo pacman -S pulseaudio-alsa  paprefs pavucontrol pasystray
```


```
kmix为图形界面音量管理
(需要先安装 alsa alsa-utils)
sudo pacman -S  kmix

启动后，可在托盘看到图标。
```


+ 设置默认声卡
```
运行 aplay -l，获取声卡的声卡ID和设备ID：
$ aplay -l
若要把这块声卡作为默认声卡，可以把下列配置添加到系统级别的 /etc/asound.conf 或用户级别的 ~/.asoundrc 文件。如果文件不存在，可以手动创建。其中的各个ID，请根据实际情况调整：
pcm.!default {
	type hw
	card 2
}
ctl.!default {
	type hw           
	card 2
}

在大部分情况下，推荐您使用声卡名来代替数字形式的参考名，同时这也能够解决启动顺序的问题。
为了获得卡的名称，任选一个下面的命令：
$ aplay -l | awk -F \: '/,/{print $2}' | awk '{print $1}' | uniq
$ cat /proc/asound/card*/id
或
$ cat /proc/asound/card*/id

编辑配置添加到系统级别的 /etc/asound.conf 或用户级别的 ~/.asoundrc 文件。
pcm.!default {
	type hw
	card Audio
}

ctl.!default {
	type hw           
	card Audio
}
```

+ 问题记录
```
设备：ASUS-AX550DP笔记本电脑
接外放设备，外放设备能播放出声音，拔掉设备接的耳机孔接入，不能切换到自己的自带的外放设备中播放。

```

> 参考资料

+ [手把手教你安装 Archlinux](https://my.oschina.net/wuzsheng/blog/1622254)
+ [ArchLinux安装完没有声音之解决办法](https://segmentfault.com/a/1190000002918394)
+ [archlinux alsa安装，音量设置和音量信息保存](https://blog.csdn.net/Sherlock955/article/details/84333838)

+ [在Linux下，有时会突然没有声音..(解决方法)_老余_新浪博客](http://blog.sina.com.cn/s/blog_5ce530a10100kyxf.html)
+ [archlinux 突然没声音了是什么情况 - V2EX](https://www.v2ex.com/amp/t/581585)

- [PulseAudio (简体中文)](https://wiki.archlinux.org/index.php/PulseAudio_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

- [树莓派archlinux音频输出折腾](https://blog.csdn.net/weixin_43352959/article/details/110249070)

- [alsa设置默认声卡](https://blog.csdn.net/lophyxp/article/details/14228735)

### 鼠标

---
```
	Linux提供两个驱动 mouse和evdev
	mouse是一般的驱动，提供鼠标的基本功能
	evdev是一款高级USB设备驱动，可以提供比常规Xorg mouse驱动更强大的功能，减少输入延时
	sudo pacman -S xf86-input-evdev
	注意，安装软件包以后，这个驱动可能不会自动加载，如果重庆Xorg也没有加载，那进行下面一步设置
	ln -s /usr/share/X11/xorg.conf.d/10.-evdev.conf /etc/X11/xorg.conf.d
	用来将edev的驱动指定为鼠标驱动，且xorg初始化的时候启动

	触摸板
	sudo pacman -S xf86-input-synaptics
	指定所使用的触摸板驱动
	ln -s /usr/share/X11/xorg.conf.d/70-synaptics.conf /etc/X11/xorg.conf.d

```

> 参考资料
+ [Arch Linux在Plasma环境下解决鼠标、触摸不灵的驱动问题](https://blog.csdn.net/zjx923759789/article/details/885464670)

### 网络连接-有线

```
	0.  插线
	1.  安装软件-网络连接命令行工具
		sudo pacman -S  dhcpcd   // 网络连接套餐
	
	2. 查看下可用的网卡
		ifconfig -a
		或者
		ip addr
		
	3. 启动网卡
		ifconfig eth0 up
		或者
		ip link set eth0 up

	4. 设置开机启动时 ，打开dhcpcd服务
		systemctl enable dhcpcd
```

> 参考资料
- [Archlinux网络配置](https://blog.csdn.net/HelloAnyones/article/details/84089232?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

### 网络连接-wifi

---
```
    方式一：命令行工具
        sudo pacman -S wireless_tools wpa_supplicant dhcpcd   // 网络连接套餐

        sudo ip addr //  查看网络设备情况
        sudo ip link set 设备名 up // 启动网络设备
        sudo wlist 设备名 scan                // 用无线设备如：wlan0扫描查看附近wifi列表
        sudo iwlist 设备名 scan | grep ESSID   // 用无线设备如：wlan0扫描查看附近wifi列表并过滤只显示wifi名
        wpa_passphrase 网络 密码 > 文件名 // 将网络和密码信息保存到一个配置文件中
            例： wpa_passphrase Xiaomi_26DE xzh250xxxx > internet.conf
                 wap_supplicant -c 文件名 -i 设备名 & // 后台挂起执行，用配置文 件的内容使用wifi设备连接
                例子： wap_supplicant -c internet.conf -i wlan0 &
                注： 如果一直在刷日志可能是密码不对，重新生成网络信息文件再次连
                接（用 ps 查看进程 kill -9 进程号 杀死进程）
        sudo dhcpcd  // 开启自动获取IP地址
        ping baidu.com   // ping 百度网址查看是否连接成功

    方式二：图形工具
		安装
			sudo pacman -S  networkmanager network-manager-applet
	
		查看NetworkManager状态
			service NetworkManager status

		临时启动
			systemctl start NetworkManager
		关闭
			systemctl stop NetworkManager

		查看是否开启启动
			systemctl is-enabled NetworkManager

		开机启动
			systemctl enable NetworkManager

		禁用开机启动
			systemctl disable NetworkManager

		打开字符网络管理的UI界面
			nmtui

			注：Edit a connect 查看或编辑已经连接过的网络信息与配置
				Activate a connection 选择与激活网络链接（可进入查看wifi列表）
```

```
	查看连接的wifi的密码
	sudo -i                                    // 提升用户权限
	cd /etc/NetworkManager/system-connections  // 进入NetworkManager
	ls                                         // 查看所有连接过的wifi
	sudo cat 指定wifi名称                      //在wifi-security字段内的psk即为密码
```

+ 问题记录：
```
执行： sudo ip link set wlp3s0 up 
提示报错：如下
RTNETLINK answers: Operation not possible due to RF-kill

原因： 不小心按了Fn+F5 或 Fn+F8 (各个笔记本型号按键不同)把无线网卡禁用
```

```
nmtui不显示wifi列表


命令查看驱动： rfkill list 
如果wifi都显示blocked为no，则已经开启，否则需要开启

```

```

~ ❯ wpa_supplicant -c topvision5G.conf -i wlp3s0 &
Successfully initialized wpa_supplicant
nl80211: deinit ifname=wlp3s0 disabled_11b_rates=0
wlp3s0: Failed to initialize driver interface
[1]  + 5038 exit 255   wpa_supplicant -c topvision5G.conf -i wlp3s0

没有使用root权限执行

```

```
nmtui nmcli

~ ❯ nmcli connection up top\ vision
错误：连接激活失败：No suitable device found for this connection (device br-cbd9b07344b7 not available because profile is not compatible with device (mismatching interface name)).


解决办法：
如果新加网卡或者克隆虚拟机，可能是 mac 地址不对，使用 ip addr 查看mac地址

永久修改：
然后把  /etc/sysconfig/network-scripts/ifcfg-ens33（MACADDR） 或者 /sys/class/net/ens33/address

临时修改：
#ifconfig eth0 down /*禁掉eth0网卡，这里以eth0网卡为例*/

#ifconfig eth0 hw ether 00:AA:BB:CC:DD:EE /*修改eth0网卡的MAC地址*/

#ifconfig eth0 up   /*重新启动eth0网卡*/

```


> 参考资料
+ [关于NetworkManger service](https://blog.csdn.net/xtggbmdk/article/details/80921606)
+ [Linux查看WiFi SSID密码的方法](http://www.xitongzhijia.net/xtjc/20170820/104845.html)
+ [Linux 网络wifi操作常用命令，查看WiFi密码](https://blog.csdn.net/qq_27413937/article/details/99714197)
+ [sudo: cd:找不到命令](https://blog.csdn.net/a1010256340/article/details/79728526)

### 博通无线网卡

1. 安装一个网络管理控制台
	- NetworkManager（推荐）

2. 再次安装（默认已经安装好了AUR助手yay）
```
sudo pacman -S linux-headers
yay -S broadcom-wl-dkms
```

3. wl 模块可能会与其他模块冲突而无法加载。加载wl模块之前， 请移除b43或者其他可能造成冲突的模块：
```
rmmod b43
```
	> 如果 ssb 加载了，也请一并移除:
```
rmmod ssb
```
	> Note: 错误的加载 ssb 可能导致无线界面无法被创建。

4. 加载 wl 模块：
```
sudo modprobe wl
```
	> 安装好驱动后，可以重启系统试试，

> 参考资料
- [archlinux是博通卡的烦恼](https://blog.csdn.net/qq_37904849/article/details/103899223)
- [archlinux bcm4360 无线网卡驱动](https://www.jianshu.com/p/21667171d2b1)
- [ArchLinux使用NetworkManager设置无线网的问题](https://blog.csdn.net/chitong8173/article/details/100726917)

### 设备挂载

---
```
    临时挂载法：
        sudo lsblk   // 查看设备及其分区列表
        sudo mount /dev/sdX  /mnt/xxx  // 把分区sdx挂载到/mnt/xxx下
        sudo umount /dev/sdX  /mnt/xxx // 卸载分区挂载

    长期挂载法：
        sudo lsblk   // 查看设备及其分区列表
        blkid        // 查看设备分区的UUID
        如：sudo blkid /dev/sdax 

        sudo vim /etc/fstab   // 打开信息配置文件
    	写入要挂载的分区的信息
		格式： UUID=123456678899        /home/windows10     ntfs  default  0 0
		此时，挂载的分区没有写权限，只有root可操作。


```

```
格式：
UUID=123456678899        /home/windows10     ntfs  default  0 0

参数说明：
	第一列：Device：磁盘设备文件或者该设备的Label或者UUID
	　　1）查看分区的label和uuid
	 　　　　Label就是分区的标签，在最初安装系统时填写的挂载点就是标签的名字。可以通过查看一个分区的superblock中的信息找到UUID和Label name。
	　　　　 例如:我们要查看/dev/sda1这个设备的uuid和label name
	　 2）使用设备名和label及uuid作为标识的不同
	        使用设备名称（/dev/sda)来挂载分区时是被固定死的，一旦磁盘的插槽顺序发生了变化，就会出现名称不对应的问题。因为这个名称是会改变的。
			不过使用label挂载就不用担心插槽顺序方面的问题。不过要随时注意你的Label name。至于UUID，每个分区被格式化以后都会有一个UUID作为唯一的标识号。使用uuid挂载的话就不用担心会发生错乱的问题了。

	第二列：Mount point：设备的挂载点，就是你要挂载到哪个目录下。

	第三列：filesystem：磁盘文件系统的格式，包括ext2、ext3、reiserfs、nfs、vfat等

	 
	第四列：parameters：文件系统的参数

	| Async/sync  | 设置是否为同步方式运行，默认为async
	| auto/noauto | 当下载mount -a 的命令时，此文件系统是否被主动挂载。默认为auto
	| rw/ro     |  是否以以只读或者读写模式挂载
	| exec/noexec | 限制此文件系统内是否能够进行"执行"的操作
	| user/nouser | 是否允许用户使用mount命令挂载
	| suid/nosuid | 是否允许SUID的存在
	| Usrquota | 启动文件系统支持磁盘配额模式
	| Grpquota | 启动文件系统对群组磁盘配额模式的支持
	| Defaults | 同事具有rw,suid,dev,exec,auto,nouser,async等默认参数的设置


	第五列：能否被dump备份命令作用：dump是一个用来作为备份的命令。通常这个参数的值为0或者1

	| 0 | 代表不要做dump备份        | 
	| 1 | 代表要每天进行dump的操作   | 
	| 2 | 代表不定日期的进行dump操作 | 

	第六列：是否检验扇区：开机的过程中，系统默认会以fsck检验我们系统是否为完整（clean）。

	| 0 |  不要检验                  | 
	| 1 |  最早检验（一般根目录会选择）| 
	| 2 |  1级别检验完成之后进行检验   |  


```
```
fstab 指定挂载的用户与权限

1. 通过设置 fmask, dmask, uid, gid参数可以控制文件目录的默认权限以及所属用户和组。

	设置: dmask=022,fmask=133
	对目录：
	​ 组和其他用户没有写权限。
	对文件：
	​ 所有用户可读，屏蔽执行权限。组和其他用户屏蔽写权限。

2. 设置挂载用户为登录用户
	执行命令：id username 来查看username的gid和uid，例如： kency换为你的用户名
	id kency
	uid=1000(kency) gid=1000(kency) 组=1000(kency),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),112(lpadmin),127(sambashare)

3. 最终配置
	UUID=DB85-D8AA /home/windows10    auto    defaults,utf8,uid=1000,gid=1000,dmask=022,fmask=133        0   0

```
> 参考资料
- [Linux命令-自动挂载文件/etc/fstab功能详解[转]](https://www.cnblogs.com/qiyebao/p/4484047.html)
- [fstab 指定挂载的用户与权限](https://www.koomu.cn/fstab-mount-with-uid-gid-fmask-dmask/)

### 触摸板

桌面用户无需安装，窗口管理器用户安装：
```
sudo pacman -S xf86-input-synaptics
```

### U盘和MTP设备: 连接android手机通过mtp传输文件

1. 手机连接数据线，接上电脑USB端口，并选择使用mtp传输文件

2. 直接安装 软件包 libmtp。仅安装 libmtp 就已经可以正确的访问设备
```
    安装mtpfs、jmtpfs、android-file-tranfer
    sudo pacman -S libmtp mtpfs
    yay -S jmtpfs android-file-tranfer
    
   
    jmtpfsAUR - 据说对 Android 4+ 以上的设备支持更好
    go-mtpfs-git - 据说对 Android 3+ 以上的设备支持更好
    simple-mtpfs
    android-file-transfer - MTP 客户端，有精简的 UI
```

3. 使用sudo jmptfs -l，看到手机的“busnum”和“devnum”

4. 建立挂在点，为了方便，我在/media下建立。使用命令sudo mkdir /media/mtp即可。

5. 改动权限，sudo chmod 775 /media/mtp

6.  挂载手机 sudo jmtpfs -device=1,8 /media/mtp/ 
注意： 此次device后面的数字在第三步中的信息中查看

7. 打开 /media/mtp/，就可以访问手机，进行文件的管理。卸载使用umount即可。

+ 其他法子

```
桌面环境一般能自动挂载。窗口管理器用户：

使用udisk和libmtp

pacman -S udiskie udevil
systemctl enable devmon@username.service    #username是用户名
pacman -S libmtp
​ 在/media目录下即可看到挂载的移动设备。

使用gvfs gvfs-mtp（xfce、lxde等桌面如果不能挂在mtp，也可安装gvfs-mtp ）
pacman -S gvfs    #可自动挂载u盘
pacman -S gvfs-mtp    #可自动挂载mtp设备
```

> 参考资料
- [Archlinux 用MTP方式访问安卓手机](https://www.cnblogs.com/zhangshaojian/p/13380589.html)
- [debian 下使用MTP（USB）连接手机](https://blog.csdn.net/naxybitfender/article/details/79310079)

### 能够挂载安卓手机: gvfs-mtp

### 护眼工具，需要额外配置：redshift

### 电源管理

---
```
    sudo pacman -S acpi

    使用：
    acpi
```

+ 电池配置
```
sudo vim /etc/systemd/logind.conf
 
修改以下参数配置
•HandlePowerKey           按下电源键后的动作
•HandleSuspendKey         按下挂起键后的动作
•HandleHibernateKey       按下休眠键后的动作
•HandleLidSwitch          笔记本合上盖子后的动作
•HandleLidSwitchDocked    如果笔记本放到了扩展坞或连接了多个显示器时，笔记本翻盖合上时触发的动作
```
选项： poweroff-关机、suspend - 挂起、hibernate - 睡眠、ignore - 忽略。
如：
```
HandlePowerKey=poweroff         按下电源键后的动作  关机
HandleSuspendKey=ignore         按下挂起键后的动作，忽略
HandleHibernateKey=ignore       按下休眠键后的动作，忽略
HandleLidSwitch=ignore          笔记本合上盖子后的动作,忽略
HandleLidSwitchDocked=ignore    如果笔记本放到了扩展坞或连接了多个显示器时，笔记本翻盖合上时触发的动作，忽略
```

修改后需要运行
```
sudo systemctl restart systemd-logind
```
使上述更改生效。

[logind.conf 中文手册](http://www.jinbuguo.com/systemd/logind.conf.html)有logind.conf参数的详细介绍。

> 参考资料
- [Archlinux电源管理](https://www.cnblogs.com/mc-r/p/11253429.html)



### 误删文件恢复方法（debugfs）

1. 查看要恢复的误删除文件所在分区
	```
	df ./
	或
	lsblk
	```

2. 启动debugfs工具
	```
	cj@cj-virtual-machine:~/Documents/debugfs_example$ sudo debugfs
	debugfs 1.44.1 (24-Mar-2018)
	debugfs:  open /dev/sda1
	debugfs:  ls -d /home/cj/Documents/debugfs_example/
	```

	若提示权限问题无法打开分区，请使用root权限打开debugfs工具。
	/home/cj/不可使用~/替代

	ls -d 后会出现如下信息，找到删除文件1.c，记录下尖括号内的数值，按q回到debugfs。
	```
	1574187  (12) .    1576545  (4072) ..   <1582211> (16) .1.c.swp   
	<1590178> (4044) 1.c   
	(END)
	```

	然后使用logdump命令，并使用quit退出debugfs如下
	```
	debugfs:  logdump -i <1590178>
	Inode 1590178 is at group 194, block 6292541, offset 128
	Journal starts at block 33979, transaction 115345
	No magic number at block 36187: end of journal.
	debugfs:  quit
	```

3. 恢复文件
	```
	cj@cj-virtual-machine:~/Documents/debugfs_example$ sudo dd if=/dev/sda1 of=/home/cj/Documents/debugfs_example/1.c bs=128 count=1 skip=629541
	1+0 records in
	1+0 records out
	128 bytes copied, 0.000390194 s, 328 kB/s
	```
	bs值为offset
	skip值为block
	此时文件恢复成功

> 参考资料
+ [linux误删文件恢复方法（debugfs）]（https://www.jianshu.com/p/90142b8a5c1b）

## 桌面

### 桌面服务

```

    pacman -S xorg xorg-server 


```


### Deepin桌面

---
```
    pacman -S xorg xorg-server 

    pacman -S lightdm           // 安装登录管理器/显示管理器
    pacman -S deepin deepin-extra  lightdm-deepin-greeter   // Deepin桌面基本包


    pacman -S file-roller evince gedit thunderbird gpicview          // 安装相关的依赖组件
    
    pacman -S unrar unzip p7zip         // 相关的依赖组件

    启动桌面
    修改登录管理器，设置为deepin登录
    vim /ect/lightdm/lightdm.conf  // 找到 greeter-session 这行去掉注释 改为greeter-session = lightdm-deepin-greeter

    sudo systemctl enable lightdm  // 设置开机启动登录管理器

    sudo systemctl start lightdm  // 现在直接启动

    sudo systemctl disable lightdm  // 设置开机不启动启动登录管理器
```

---
```
    sudo pacman -S ntfs-3g     // ntfs
    df -h
    sudo ntfsfix /dev/sdb2     //解除deepin ntfs格式磁盘只读

```

> 参考资料

+ [Arch Linux桌面环境美化（Xfce4）macOS like](https://www.jianshu.com/p/99f15b7ea83d)
+ [从 macOS 到 Arch Linux](https://blog.gimo.me/post/from-macos-to-archlinux/)
+ [Linux平铺窗口管理器:i3,sway,Qtile,dwm,awesome,附安装方法](https://ywnz.com/linuxrj/3403.html)
+ [适用于Linux的10个最佳平铺窗口管理器](https://www.howtoing.com/best-tiling-window-managers-for-linux)

+ [Arch LInux安装dde（Deepin Desktop Environment 深度桌面环境）](https://www.cnblogs.com/JJUT/p/9956348.html)

### KDE桌面

+ 基本安装
```
	在安装 Plasma 之前，请确保 Xorg 已经被安装到您的系统中并可以正常工作
		sudo pacman -S xorg
	安装KDE
		首先安装plasma包
    		sudo pacman -S plasma
		安装 kde下的控制台终端
			sudo pacman -S konsole
		安装kde下的文件管理器
			sudo pacman -S dolphin
		如果要使用完整的kde应用程序的话，还需要安装kde-applications包
			sudo pacman -S kde-applications
	再安装中文语言文件包
		sudo pacman -S  kde-l10n-zh_cn
		安装wqy-zenhei和wqy-microhei
		sudo pacman -S wqy-zenhei wqy-microhei
	配置显示管理器sddm
		sudo pacman -S sddm
		安装完桌面之后，就可以配置显示管理器了。首先需要生成一个配置文件，
		sudo sddm --example-config >/etc/sddm.conf
```

+ 美化配置
```

安装yakuake下拉式终端
sudo pacman -S yakuake


安装文件管理器
sudo pacman -S nautilus    //并尽可能安装可选依赖

	nautilus自动挂载window分区免除密码配置

	$ sudo vim /etc/polkit-1/rules.d/49-nopasswd_global.rules
	[然后输入]
	/* Allow members of the wheel group to execute any actions
	 * without password authentication, similar to "sudo NOPASSWD:"
	 */
	polkit.addRule(function(action, subject) {
	    if (subject.isInGroup("wheel")) {
	        return polkit.Result.YES;
	    }
	});

安装 vscode
yay -S visual-studio-code-bin

安装字体
yay -S wqy-microhei wqy-microhei-lite wqy-zenhei wqy-bitmapfont
yay -S noto-fonts-sc  

系统信息查看
sudo pacman -S neofetch   //并尽可能安装可选依赖

安装录屏/录音软件
$ sudo pacman -S obs-studio   //并尽可能安装可选依赖
$ sudo pacman -S ffmpeg    //并尽可能安装可选依赖

安装远程桌面连接控制
yay -S teamviewer   //并尽可能安装可选依赖


安装网络分析器wireshark
sudo pacman -S wireshark-qt  //并尽可能安装可选依赖
sudo gpasswd -a $(whoami) wireshark   //不用sudo权限即可抓网卡

安装微信
yay -S deepin.com.wechat2    //并尽可能安装可选依赖

安装程序坞
sudo pacman -S latte-dock

```

+ 主题设置
```
1.实现左上角的苹果菜单
这个功能可是让我找了好久…

1、右键顶栏，添加部件，获取新部件，下载新plasma部件
2、搜索“USwitcher”安装，放到左上角。
3、右键USwitcher配置，选择“only show icon”，把图标改成苹果图标。

2.实现mac的launchpad
1、添加部件，下载新plasma部件（方法同上）
2、搜索“UMenu”下载安装，自己把图标配置成launchpad的样子。
3、把UMenu部件拖到latte dock上

3、实现mac的程序名顶栏显示
1、安装插件application title（方法同上）
2、放对位置

4、我的插件布局：
USwitcher – applications icon – 全局菜单 – 面板间距 – InLine battery – 系统托盘 – event calendar – 用户切换器 – 查找
```

+ 主题手动离线安装(本次kde版本为KDE5)
```
第一步，下载 ocs-url 通过 yay -S ocs-url下载;
	[kde主题网站](https://store.kde.org/)
	[](https://store.kde.org/browse/cat/104/ord/top/)

第二步，点击主题边上的 install按钮，之后会跳出是否打开 xdg-open，点击打开xdg-open,然后就会调用 ocs-url 下载安装主题，ocs-url可以自动下载主题，安装主题，同样适用于插件、plasma等;

另外：如果不使用以上方法，你也可以点击主题旁边的”download“按钮，下载对应的压缩包，解压之后移动到对应的目录：

￼
/home/hzt/.local/share/plasma/desktoptheme 这是存放plasma主题
/home/hzt/.local/share/plasma/look-and-feel/ 存放全局主题
/home/hzt/.local/share/plasma/plasmoids/ 存放插件
以上目录如果没有就自行创建。


1. 下载全局主题。

选择：Global Themes ,选择合适的主题下载。

下载完成后，解压到~/.themes文件夹中，如果没有这个文件夹可以手动创建一个。

2. 下载鼠标(光标)主题。

选择：Cursors，选择下载

将解压后的文件同样放在～/.themes中。

3. 下载图标文件文件。

选择：Full Icon Themes，下载喜欢主题。

解压文件放到~/.icons文件夹中。

4. Latte-dock配置文件 

选择：Latte Dock，下载文件。

解压文件到：～/.config/latte/中。
```

+ KDE中快捷键
```
系统设置--快捷方式和手势--全局键盘快捷键--kwin里最后俩

卷屏，ON/OFF，C-F2/C-F1

显示桌面，这个跟WIN的又不一样，桌面临时置顶，并不会把所有程序最小化，C-F12

显示程序，类似Gnome3里边SUPER-W的那种显示方式，C-F7/C-F9/C-F10

切换桌面，C-F8/C-F11

程序启动器，类似Win里边WIN+R的效果，所不同的是这个启动器会自己搜索PATH里面的可执行文件，A-F2

右键K按钮，编辑程序，可以为任意程序设置启动快捷键和启动参数

文件浏览器里面按F4是可以打开终端的，按F3是可以像VIM那样分屏显示的，按F11可以随时去掉文件信息框的

按A-F4可以向当前的GUI进程发送TERM信号，但不是KILL，目测这个信号可以拦截下来

ctl+tab 切换工作桌面
alt+tab 切换应用程序
alt+f1 打开应用程序菜单
alt+f2 打开命令窗口
alt+f3 打开窗口菜单
alt+f4 关闭窗口
f1 帮助
f2 在页面中搜索
f3 在页面中查找下一个
space 选中/取消选中 文件
ctl+n 打开文件管理器
ctl+a 全选当前目录下的文件和目录
ctl+t 在当前目录下打开一个终端
ctl+w 关闭窗口
ctl+f 搜索文件
ctl+c 拷贝
ctl+v 粘贴

绚丽的桌面效果。可以使用 Ctrl+F8 开启或者 Ctrl+F9

启动系统活动工具窗口（任务管理器） Ctrl+Esc 
```


+ 问题记录
```
终端中中文文字不显示或乱码现象

解决:
	export LANG=zh_CN.UTF-8
	发现一部分内容汉化了
	修改成
	export LC_ALL=zh_CN.UTF-8
	重启系统
```

	> 参考资料
	- [KDE如何配置得漂亮大气？KDE如何配置得漂亮大气？](https://www.zhihu.com/question/54147372/answer/150096870)
	- [Manjaro-KDE配置全攻略](https://zhuanlan.zhihu.com/p/114296129)
	- [[ArchLinux] 安装及KDE桌面环境安装配置](https://blog.csdn.net/hepangda/article/details/82817997)
	- [Arch Linux 2020-07 安装kde桌面环境](https://www.jianshu.com/p/5e7726d1cb16)
	- [[笔记]manjaro kde主题&手动安装主题](https://www.cnblogs.com/hztjiayou/p/12054384.html)
	- [KDE桌面美化](https://www.cnblogs.com/mc-r/p/12219389.html)
	- [KDE仿mac](https://blog.csdn.net/weixin_44402295/article/details/103434529?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduend~default-2-103434529.nonecase&utm_term=kde%E4%BB%BFmac%E7%BE%8E%E5%8C%96&spm=1000.2123.3001.4430)
	- [5. KDE 桌面环境](https://opensuse-guide.ustclug.org/kde.php)

	- [KDE的若干快捷键，备忘](https://www.cnblogs.com/huyc/archive/2012/11/24/2785704.html)

	- [KDE部分中文乱码](https://blog.csdn.net/baigang1986/article/details/102335251)

### Xfce4桌面

```
		pacman -S lightdm 

		pacman -S xfce4 xfce4-goodies  // Xfce4基本包

		pacam -S  lightdm-gtk-greeter //


		启动桌面
		修改登录管理器，设置为gtk登录
		vim /ect/lightdm/lightdm.conf  // 找到 greeter-session 这行去掉注释 改为greeter-session = lightdm-gtk-greeter
		
		sudo systemctl enable lightdm  // 设置开机启动登录管理器

		sudo systemctl start lightdm  // 现在直接启动

		sudo systemctl disable lightdm  // 设置开机不启动启动登录管理器

		注：如果安装多个桌面在登录界面的右下角可以选择左面切换
```


```

	下载安装主题
		https://www.pling.com/p/1326265/ 

	两个文件：
 
	McOS-CTLina-XFCE.tar.xz（浅色）

	Mc-OS-CTLina-XFCE-Dark.tar.xz（深色）

	解压主题到以下目录(没有的话可创建一个)

	/usr/share/themes/（系统范围有效，需要管理员权限）

	~/.themes/（用户范围有效，别的用户不能用）

	设置主题
	打开Settings Manger（设置管理器）界面

	Appearance设置
	打开Appearance（外观）

	1.切换到刚才下载的主题

	Window Manager设置
	打开Window Manager（窗口管理器）

	1. 切换Theme（主题）到刚才下载的主题

	2. 在Button Layout（按钮布局）区域，拖动窗口按钮到左边（关闭，最小化，最大化）

	Window Manager Tweak 设置
	打开Window Manager Tweak（窗口管理器微调）

	1. 切换到Compositor（合成器）界面

	2. 取消选中Show shadows under dock windows（在dock窗口下显示阴影）

	3. 设置一些透明度

	设置Panel（面板）
	1. 右键Panel（面板），找到Panel Perferences（面板首选项）

	2. 取消选中Lock Panel（锁定面板）

	3. 拖动面板到屏幕顶端
	设置Dock停靠栏
	安装
	使用命令安装Plank（或者从应用中心安装）：

	sudo pacman -S plank

	安装后可在应用程序里面启动（或者使用命令plank启动）

	开机启动
	在设置管理器里面找到Session and Startup（会话和启动），在Application Autostart（应用程序自启动）里面点击Add（添加）按钮，新增Plank登录自启动。

```

> 参考资料

+ [Archlinux安装xfce4桌面及美化流程](https://blog.csdn.net/kingolie/article/details/76723448)
+ [Arch Linux桌面环境美化（Xfce4）macOS like](https://www.jianshu.com/p/99f15b7ea83d)

### 桌面：dwm

---
```
    sudo pacman -S xorg xorg-server  xorg-xinit      // 依赖
    sudo pacman -S dmenu  // X下轻量级的动态菜单

    安装方式一：源码编译安装
        1、下载源码
            git clone git://git.suckless.org/dwm
        2、编辑修改（可选）
            
        3、编译打包
            a、直接手动make编译打包
                make                // 编译
                make clean install  // 安装
            b、ArchLinx的编译打包规范方式打包 

    安装方式二：
        sudo pacman -S dwm

    多屏幕控制
    安装相关软件所需
    sudo pacman -S xorg-xrandr
   
    1. 查看设备
    	xrandr
   
    输入xrandr，查看输出中状态是connected的显示设备，如LVDS。具体命令可以是：
    	sudo xrandr | grep -v disconnected | grep connected
  
    1. 调整亮度：
		# xrandr —output LVDS —brightness 0.5
		注：output后面的参数为上一步中查出的显示设备，不同主机结果可能不同。brightness后面的参数范围是0-1，0为全黑，1为最亮。

		如果您明确知道你的分辨率的话，你可以将这个参数直接写成你需求的分辨率，如下：
		# xrandr -s 1024×768

		也可以使用 -q 参数来查看你的屏幕目前支持的分辨率的情况，或者什么参数也不加。
		# xrandr -q
		# xrandr

		当然这个命令还有一些更复杂的用法，您可以用 info 命令来查看：
		# info xrandr 

		命令
		xrandr --output VGA --same-as LVDS --auto  
		打开外接显示器(最高分辨率)，与笔记本液晶屏幕显示同样内容（克隆）

		命令
		xrandr --output VGA --same-as LVDS —mode 1024x768  
		打开外接显示器(分辨率为1024x768)，与笔记本液晶屏幕显示同样内容（克隆）

		命令
		xrandr --output VGA --right-of LVDS --auto  
		打开外接显示器(最高分辨率)，设置为右侧扩展屏幕

		命令
		xrandr --output VGA --off  
		关闭外接显示器

		命令
		xrandr --output VGA --auto —output LVDS --off  
		打开外接显示器，同时关闭笔记本液晶屏幕（只用外接显示器工作）

		命令
		xrandr —output VGA --off --output LVDS --auto  
		关闭外接显示器，同时打开笔记本液晶屏幕 (只用笔记本液晶屏)
		（最后两种情况请小心操作，不要误把两个屏幕都关掉了....）
```
> 参考资料
- [xrandr命令](https://blog.csdn.net/wangzhen209/article/details/45218825)
- [记录一下xrandr命令](https://blog.csdn.net/kendyhj9999/article/details/6427617?utm_medium=distribute.wap_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase&depth_1-utm_source=distribute.wap_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase)
- [dwm配置多显示器](https://blog.csdn.net/u010563350/article/details/106732434)


---
```
    问题记录
    makepkg -g >> 
    报错 :
    ==> ERRPR: pkhgrel is not allowed to be empty.

    ==> ERRPR: PKGBUILD does not exit.
```

+ [Archlinux 的灵魂──PKGBUILD、AUR 和 ABS](https://blog.csdn.net/taiyang1987912/article/details/41457333)


### 桌面：i3、i3wm


```
    安装:
        sudo pacman -S xorg xorg-server xorg-init
        sudo pacman -S i3-wm i3status i3blocks i3lock 
         
    配置:
        配置文件位置~/.config/i3/config
        在 i3 里，一切命令均以「修饰键」开头，即 $mod. 默认上来说是 Alt 键
        alt(Mod1),但开始键 Super(Mod4) 也更为广泛接受。Super 键往往带有 Windows
        图标

    启动与退出
        方式一：从xinitrc启动
            exec i3

```

+ 参考案例
- [i3wm桌面配置案例1:prettyi3](https://github.com/aeghn/prettyi3)
- [i3wm桌面配置案例2:prettyi3](https://gitee.com/endlesspeak/prettyi3)

> 参考资料

- [i3桌面官方指南手册](https://i3wm.org/)

+ [i3 (简体中文)](https://wiki.archlinux.org/index.php/I3_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
+ [i3窗口管理器入门](https://linux.cn/article-10021-1.html)

- [装b新技能，执手(鼠)画终端——archlinux,i3演示第二季](https://www.bilibili.com/video/BV1x7411c7tG?from=search&seid=7919922028367493020)

- [Arch Linux: 从0配置到比Manjaro还好用](https://www.bilibili.com/video/BV1EK4y1E7Rd/?spm_id_from=trigger_reload)

- [Archlinux下i3wm与urxvt的配置](https://www.cnblogs.com/vachester/p/5649813.html)

- [一份自用的简单i3wm配置](https://segmentfault.com/a/1190000022083424)

### 桌面： Pantheon 

Pantheon 是linux发行版 elementary os 的默认桌面环境。由开发者使用vala语言和gtk3工具包编写完成，高效并且易于使用。用户界面上，与GNOME-shell和Mac OS X多有相似之处。

+ 安装
```
yay -S pantheon pantheon-session-git gala plank
```

+ 通过 .xinitrc 启动
> ~/.xinitrc 启动Pantheon。这段代码将能够成功启动Pantheon:
```
#!/bin/sh
 
if [ -d /etc/X11/xinit/xinitrc.d ]; then
  for f in /etc/X11/xinit/xinitrc.d/*; do
    [ -x "$f" ] && . "$f"
  done
  unset f
fi

gsettings-data-convert &
xdg-user-dirs-gtk-update &
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
/usr/lib/gnome-settings-daemon/gnome-settings-daemon &
/usr/lib/gnome-user-share/gnome-user-share &
eval $(gnome-keyring-daemon --start --components=pkcs11,secrets,ssh,gpg)
export GNOME_KEYRING_CONTROL GNOME_KEYRING_PID GPG_AGENT_INFO SSH_AUTH_SOCK
exec cerbere
```


+ 桌面环境
	- gala：橱窗和堆肥经理
	- wingpanel：顶部面板（容纳发射器，时钟和指示器）
	- pantheon-applications-menu：应用程序启动器，以前称为“ Slingshot”
	- plank：macOS样式的Dock程序扩展坞

+ 主题和配置（这些可选软件包有助于桌面的外观和感觉）：
	- lightdm-p​​antheon-greeter：LightDM迎宾员
	- pantheon-default-settings AUR：默认外观，行为和配置；拉出主题包和字体：
		- elementary-icon-theme：基本元素起源的矢量图标主题
		- elementary-wallpapers：基本操作系统壁纸集合
		- gtk-theme-elementary：基本操作系统样式表
		- ttf-droid：来自Google Android的通用字体
		- ttf-opensans：Google的Sans-serif字体
		- ttf-roboto：Google的签名字体家族
	- sound-theme-elementary：声音主题基础，一组系统声音
	- switchboard：可插拔的设置管理器，类似于gnome-control-center

+ 应用领域（以下是一些原始的，修补的和选定的软件包，其中包括可选的基本OS软件套件）：
	- capnet-assist：轻松登录公共WiFi网络
	- epiphany：Web浏览器替换了midori-granite AUR
	- pantheon-calculator：计算器
	- pantheon-calendar：日历应用程序，以前称为“ Maya”，与wingpanel-indicator-datetime集成
	- pantheon-camera：以前称为“ Snap”的网络摄像头应用
	- pantheon-code：文本编辑器，以前称为“ Scratch”
	- pantheon-files: 从马林开发的文件浏览器
	- pantheon -mail AUR：由Geary开发的电子邮件客户端，正在完全重写
	- pantheon-music：音频播放器，以前称为“噪声”
	- pantheon-photos: 照片管理器由Shotwell开发
	- pantheon-screencast: eidete-bzr AUR派生的简单截屏器
	- pantheon-screenshot: 屏幕截图实用程序
	- pantheon-shortcut-overlay：操作系统范围内的快捷键叠加层
	- pantheon-terminal:终端仿真器
	- pantheon-videos：视频播放器，以前称为“受众”（GStreamer后端）
	- simple-scan：简单扫描实用程序

+ 推荐安装以下字体包来获取最佳桌面体验:
	- ttf-opensans：Open Sans字体
	- ttf-raleway-font-family AUR [替代的链接：找不到软件包]：Raleway字体家族
	- ttf-dejavu：基于Bitstream Vera字体的字体家族
	- ttf-droid：Google作为Android的一部分发布的通用字体
	- gnu-free-fonts：一组覆盖Unicode字符集的免费轮廓字体
	- ttf-liberation：Red Hats Liberation字体

> 参考资料
- [Pantheon (简体中文)](https://wiki.archlinux.org/index.php/pantheon_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [Pantheon](https://wiki.archlinux.org/index.php/Pantheon)

## 桌面：gnome

+ 安装
```
sudo pacman -S gnome
```

+ 配置或美化
```
标题栏按钮重新排序
设置 GNOME 窗口管理器顺序 (Mutter, Metacity):
默认[设置按钮在右侧]：
	gsettings set org.gnome.desktop.wm.preferences button-layout ':minimize,maximize,close'
设置按钮在左侧
	gsettings set org.gnome.desktop.wm.preferences button-layout 'close,maximize,minimize:'
提示： 冒号表示窗口标题栏的按钮会出现在哪一方


标题栏高度：
Note: 下面配置会修改 GNOME 终端和 Chromium 的标题栏高度，但是不会影响 Nautilus。
~/.config/gtk-3.0/gtk.css
headerbar.default-decoration {
 padding-top: 0px;
 padding-bottom: 0px;
 min-height: 0px;
 font-size: 0.6em;
}

headerbar.default-decoration button.titlebutton {
 padding: 0px;
 min-height: 0px;
}


状态栏透明或颜色设置：
sudo vim /usr/share/gnome-shell/theme/gnome-shell.css

用搜索命令：/panel

应该会找到下面这一段

#panel {

color: #ffffff;
background-color: black;
border-image: url("panel-border.svg") 1;
font-size: 10.5pt;
font-weight: bold;
height: 1.86em;
}

把他改为：

#panel {
    color: #ffffff;
    background-color: rgba(0,0,0,0.3);
  /*  background-color: black;
    border-image: url("panel-border.svg") 1;*/
    font-size: 10.5pt;
    font-weight: bold;
    height: 1.86em;
}

0.3是透明度，你可以设置为其他值。

按alt+f2,输入r 回车，你就会看到效果

注: 2021.06.21 没设置成功
```

```
菜单图标
默认的GNOME设置不在菜单上显示图标。要在菜单上显示图标，运行以下命令：

$ gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "{'Gtk/ButtonImages': <1>, 'Gtk/MenuImages': <1>}"
```

+ 设置自动切换壁纸脚本
```
#!/bin/bash

# # 执行入口 
# # 进入脚本目录位置
# cd `dirname $0`
# # 获取当前脚本目录位置
# shellfilepath=`pwd`'/'$0
# # 打印脚本目录地址
# echo "[INFO]: 脚本地址 "${shellfilepath}


# 设定壁纸目录路径：
#DIR=/usr/share/backgrounds
#DIR=/home/whoami/Pictures/wallpapers
DIR=$HOME/Pictures/wallpapers

# 设定切换桌面背景的时间间隔，单位为'秒'：
SEC=30


# 判断桌面环境
# if [ "$DESKTOP_SESSION" == "gnome" ] || [ "$XDG_CURRENT_DESKTOP" == "GNOME" ] ; then 
# 	echo "当前是gnome桌面";
# else
# 	echo "当前不是gnome桌面";
# fi

# 判断脚本是否已经再执行
#sh_process=`ps -aux|grep wallpaper-autochange-gnome.sh| grep -v grep|grep -v PPID|awk '{print $2}'`
sh_process=`pidof -x "wallpaper-autochange-gnome.sh"`
for i in $sh_process
do
	# echo $i
	# echo $$
	if [ $i == $$ ] # 当前进程则跳过
	then
		continue
	fi
	# 非当前则，杀死
	kill -9 $i
done

# if pidof -x "wallpaper-autochange-gnome.sh" >/dev/null; then
# 	# 脚本已有在执行
# 	echo "Process already running"
# 	# 杀死正在执行的脚本进行
# else
# 	# 脚本未在执行
# 	echo "Process not running"

# fi

# 进入一个死循环
while true
do
	# 随机取文件夹下的一个文件
	# echo $(ls $DIR/* | shuf -n1)
	#WALLPAPER_PIC=$(ls $DIR/*.jpg | shuf -n1)
	WALLPAPER_PIC=$(ls $DIR/* | shuf -n1)
	# 设置壁纸
	#gsettings set org.gnome.desktop.background picture-uri 'file:///home/whoami/Pictures/wallpapers/wallhaven-0jy6wp.jpg'
	gsettings set org.gnome.desktop.background picture-uri 'file://$WALLPAPER_PIC'
	cmdStr="gsettings set org.gnome.desktop.background picture-uri  'file://$WALLPAPER_PIC'"
	# echo "执行命令: ${cmdStr} "
	${cmdStr}

	# 休息三分钟
	#sleep 3m
	# 休息三秒
	#sleep 3s
	sleep "$SEC"
done




设置为开机执行则新建一个 xx.Desktop 内容如下：[拷贝到 ~/.config/autostart中]

[Desktop Entry]
Name=wallpaper-autochange-gnome
GenericName=A descriptive name
Comment=Some description about your script
Exec=/home/whoami/.config/i3/scripts/wallpaper-autochange-gnome.sh
Terminal=false
Type=Application
X-GNOME-Autostart-enabled=true

ln -s /home/whoami/.config/i3/scripts/wallpaper-autochange-gnome.desktop /home/whoami/.config/autostart

或者 放到 /usr/share/applications 中
打开 gnome-tweak (安装：sudo pacman -S gnome-tweak-tool )即 优化 在自动启动程序中添加 wallpaper-autochange-gnome 则进入桌面就就会启动

```


+ 扩展(gnome-shell-extensions)
```
安装gnome-shell gnome-tweaks gnome-tweak-tool :
    yay -S gnome-shell gnome-tweaks
```

PS: gnome3.34版本及以上,extensions移出gnome-tweaks
```
安装flathub 在使用flathub安装扩展(gnome-shell-extensions):
    https://flatpak.org/setup/Arch/
	sudo pacman -S flatpak

安装:GNOME Extensions
    flatpak install flathub org.gnome.Extensions
运行:
    flatpak run org.gnome.Extensions
```

+ 扩展:dash-to-dock安装
```

git clone -b ewlsh/gnome-40 https://github.com/ewlsh/dash-to-dock.git
git clone https://github.com/ewlsh/dash-to-dock.git ./dash-to-dock-master
cp ./dash-to-dock-master/stylesheet.css ./dash-to-dock/
rm -r ./dash-to-dock-master
cd dash-to-dock
make
make install


Alt+F2 r Enter
```

+ 截图工具（screenshot-tool）插件：gnome-shell-screenshot
> 插件地址：https://extensions.gnome.org/extension/1112/screenshot-tool/
```
git clone https://github.com/OttoAllmendinger/gnome-shell-screenshot.git
cd gnome-shell-screenshot
make update_dependencies
make install

注：安装完重启gnome 后到 扩展 设置中打开 安装完默认是关闭的
```

+ 大小写状态显示:gnome-shell-extension-lockkeys
```
git clone https://github.com/kazysmaster/gnome-shell-extension-lockkeys
git clone https://hub.fastgit.org/kazysmaster/gnome-shell-extension-lockkeys
cd gnome-shell-extension-lockkeys
cp -R lockkeys@vaina.lt  ~/.local/share/gnome-shell/extensions
```


+ 启动时没有概览插件：
> 插件介绍地址：https://extensions.gnome.org/extension/4099/no-overview/
```

```

+ 声音输入输出扩展插件：sound-output-device-chooserkgshank.net.v38.shell-extension
```
git clone  https://github.com/kgshank/gse-sound-output-device-chooser
cp -r gse-sound-output-device-chooser/sound-output-device-chooser@kgshank.net   ~/.local/share/gnome-shell/extensions

```

+ 将图标加到桌面
https://extensions.gnome.org/extension/1465/desktop-icons/
```
 https://gitlab.gnome.org/World/ShellExtensions/desktop-icons

```

+ 管理剪贴板
```
https://extensions.gnome.org/extension/779/clipboard-indicator/
```

+ 用于访问和卸载可移动设备的状态菜单:Removable Drive Menu


+ Hide Top Bar:隐藏顶部栏，除了在概览中。但是，有一个选项可以在鼠标指针接近屏幕边缘时显示面板。
```
cd ~/.local/share/gnome-shell/extensions/
git clone https://github.com/mlutfy/hidetopbar.git hidetopbar@mathieu.bidon.ca
cd hidetopbar@mathieu.bidon.ca
make

```

+ 应用切换特效插件
```
https://extensions.gnome.org/extension/97/coverflow-alt-tab/
```

+ 
```
https://github.com/gfxmonk/gnome-shell-scroll-workspaces

```

+ 工作空间
```
https://extensions.gnome.org/extension/21/workspace-indicator/
```

+ 网络速度显示
```
https://extensions.gnome.org/extension/104/netspeed/
```

+ 添加菜单以快速导航系统中的位置
```
https://extensions.gnome.org/extension/8/places-status-indicator/
```

+ Lunar Calendar 农历
```
https://extensions.gnome.org/extension/675/lunar-calendar/
```

+ 时间位置插件移动
```
https://extensions.gnome.org/extension/2/move-clock/
```

+ 顶部栏和应用标题栏按钮配合
```
https://extensions.gnome.org/extension/1287/unite/
```

+ 性能监控：
> 地址: https://extensions.gnome.org/extension/1460/vitals/
```
项目地址：https://github.com/corecoding/Vitals
需要依赖
sudo pacman -Syu libgtop lm_sensors gnome-icon-theme-symbolic

git clone https://github.com/corecoding/Vitals.git ~/.local/share/gnome-shell/extensions/Vitals@CoreCoding.com

```

+ 问题记录
```
一个命令禁用baloo_file及baloo_file_extractor
命令就行：    balooctl disable
```

> 参考资料
- [ArchLinx gnome](https://wiki.archlinux.org/title/GNOME_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [GNOME 桌面必备扩展（GNOME Shell Extensions）](https://www.cnblogs.com/keatonlao/p/12686234.html)
- [2020 gnome 桌面插件推荐](https://zhuanlan.zhihu.com/p/270645604?native.theme=1)
- [gnome 40 的"dash-to-dock"解决方案](https://zhuanlan.zhihu.com/p/369561330)
- [# 2021-01-15 #「GNOME」- 扩展（Extension）](https://blog.csdn.net/u013670453/article/details/112647300)

## 软件

### 最轻量dock:plank

```
安装：
	yay -S plank

打开设置：
	plank --preferences
	
```

### SSH远程连接：ssh

---
```

    安装
        sudo pacman -S openssh

	使用
		立即开启：         systemctl satrt   sshd
		立即停止：         systemctl stop    sshd
		开机启动：         systemctl enable  sshd
		关闭开机自启动：   systemctl disable ssd

```

### ssh远程传输文件：scp
---
```
	安装
		sudo pacman -S openssh-server
		sudo pacman -S openssh-clients

```

### 默认程序打开文件：xdg-open

```
	xdg-open filename.png
	会使用默认的png查看器打开图片

	xdg-open filename.mp4 
	会使用默认的png查看器打开图片

```

### 远程桌面连接：xrdp

---

1. 安装（被连接）
```
	aurman -S xrdp  或 yay -S xrdp
	sudo pacman xorg-xrdb libdrm tigervnc  xterm
```

1. 配置(桌面为dwm)
```
	1. 首先需要修改一下xrdp监听的端口，xrdp默认是监听在3389端口上，
		若是Windows的Linux子系统，需要更改端口号，因为windows主机和linux子系统是在同一台机器上，因此windows自己的远程桌面程序监听的3389端口就会和linux子系统上的xrdp监听的3389端口冲突。我们先打开xrdp程序的配置文件，命令是：
		//备份
		sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
		//修改
		sudo vim /etc/xrdp/xrdp.ini

	
	1. 将dwm的会话环境写入到默认的会话环境配置文件中去，命令是：
		sudo echo dwm > ~/.xsession
		
	1. 配置xrdp
		sudo vim /etc/xrdp/startwm.sh
		在约71行附近  #arch  if [ -r /etc/X11/xinit/xinitrc ];then 的下一行  ./etc/X11/Xsession 的 前一行插入
		dwm
		
	1. 配置vnc：
		VNC 服务读取 ~/.vnc/xstartup 文件（功能类似于 .xinitrc）。如果需要图形环境，则用户至少需要定义一个桌面环境来启动。例如：启动dwm

		#!/bin/sh
		export XKL_XMODMAP_DISABLE=1
		exec startx dwm
```

1. 	启动
```
	1. 启动xrdp服务，命令是：
		sudo systemctl start xrdp xrdp-sesman
		sudo /etc/xrdp/startwm.sh

	1. 查看IP地址
		ifconfig
		或
		ip addr
		
	1. 在另一台电脑上使用远程登录连接(例如用Windows的远程桌面,选择vnc)
	
	1. 查看xrdp日志/var/log/Xorg.0.log 、/var/log/xorg.log、/var/log/xorg-sesman.log
	

```

1. 设置开机自动启动
```
	1. 启动xrdp服务，命令是：
		sudo systemctl enable xrdp xrdp-sesman
		sudo /etc/xrdp/startwm.sh 

```

1. 重启 xrdp
```
	sudo systemctl restart xrdp xrdp-sesman
```

+ 遇到问题记录

	+ 登录不了到桌面 报错信息 [DEBUG]Closed socket 18 ...
```
	[20200409-16:53:42] [DEBUG] Security layer: requested 11, selected 1
	[20200409-16:53:42] [INFO ] connected client computer name: CodeRooster
	[20200409-16:53:42] [INFO ] adding channel item name rdpdr chan_id 1004 flags 0x80800000
	[20200409-16:53:42] [INFO ] adding channel item name rdpsnd chan_id 1005 flags 0xc0000000
	[20200409-16:53:42] [INFO ] adding channel item name cliprdr chan_id 1006 flags 0xc0a00000
	[20200409-16:53:42] [INFO ] adding channel item name drdynvc chan_id 1007 flags 0xc0800000
	[20200409-16:53:42] [INFO ] TLS connection established from 192.168.124.15 port 62912: TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384
	[20200409-16:53:42] [DEBUG] xrdp_0001df53_wm_login_mode_event_00000001
	[20200409-16:53:42] [INFO ] Loading keymap file /etc/xrdp/km-00000409.ini
	[20200409-16:53:42] [WARN ] local keymap file for 0x00000409 found and doesn't match built in keymap, using local keymap file
	[20200409-16:53:42] [DEBUG] xrdp_wm_log_msg: connecting to sesman ip 127.0.0.1 port 3350
	[20200409-16:53:46] [DEBUG] Closed socket 18 (AF_INET 127.0.0.1:45000)
	[20200409-16:53:50] [DEBUG] Closed socket 18 (AF_INET 127.0.0.1:45032)
	[20200409-16:53:54] [DEBUG] Closed socket 18 (AF_INET 127.0.0.1:45064)
	[20200409-16:53:58] [ERROR] xrdp_wm_log_msg: Error connecting to sesman: 127.0.0.1 port: 3350
	[20200409-16:53:58] [DEBUG] Closed socket 18 (AF_INET 127.0.0.1:45096)
	[20200409-16:53:58] [DEBUG] return value from xrdp_mm_connect 1
	[20200409-17:00:55] [DEBUG] Closed socket 12 (AF_INET 192.168.124.10:3389)
	[20200409-17:00:55] [DEBUG] xrdp_mm_module_cleanup

	解决：
	sudo pacman -S xorg-xrdb
	sudo pacman -S libdrm

	使用vncserver
	(安装vnc)
	sudo pacman -S tigervnc  

	（查看已安装的vnc套件）
	pacman -Ss vnc 

	配置vnc：
		VNC 服务读取 ~/.vnc/xstartup 文件（功能类似于 .xinitrc）。如果需要图形环境，则用户至少需要定义一个桌面环境来启动。例如：启动dwm

		#!/bin/sh
		export XKL_XMODMAP_DISABLE=1
		exec startdwm


	启动
	sudo systemctl start xrdp xrdp-sesman
	vncserver
```
	
```
	connecting to sesman ip 127.0.0.1 port 3350
	sesman connect ok
	sending login info to session manager,please wait...
	login successfully for display 13
	started connecting
	connection problem,giving up
	some problem
```
	
```
	connecting to sesman ip 127.0.0.1 port 3350
	sesman connect ok
	sending login info to session manager,please wait...
	login successfully for display 0
	VNC started connecting
	VNC connecting to 127.0.0 5910
	VNC tcp connected 
	VNC error - problem connecting
	some problem
```
	
```
	connecting to sesman ip 127.0.0.1 port 3350
	sesman connect ok
	sending login info to session manager,please wait...
	login faild for display 0
	先确认密码是对的。
	有使用 pam 吗？如果有检查 pam 配置
	有使用 selinux 吗？如果有，建议关闭 selinux

	可以在 /var/log 目录找 xrdp 的日志看日志里面有什么错误。

	试试为root用户设置VNC密码：
	vncpasswd root
```
	

	
	> 参考资料
	- [配置xorg的xrdp | 大专栏](https://www.dazhuanlan.com/2020/02/02/5e3627012dfcf/)
	- [Virtual Network Computing (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/Virtual_Network_Computing_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
	- [Linux xrdp远程桌面连接声音重定向 - 简书](https://www.jianshu.com/p/6655fc3fcff1?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=weixin)

	- [Linux和Windows间的远程桌面访问](https://cloud.tencent.com/developer/article/1435807)
	- [ubuntu18.04.2 xrdp 远程桌面连接黑屏 connection problem](https://blog.csdn.net/dstjwjw/article/details/89517333)
	- [14_使用windows自带的远程桌面mstsc连接Centos7.x远程桌面](https://www.jianshu.com/p/63dce85dc958)
	- [xrdp实现windows远程桌面连接问题](https://my.oschina.net/chaoshu/blog/677139)
	- [XRDP拒绝登录](https://qastack.cn/superuser/1264096/xrdp-rejecting-login)
	
	- [Ubuntu 16.04 + xrdp + Xfce 实现 Windows 远程桌面连接 Linux 配置及使用中出现的问题](https://blog.csdn.net/yyywxk/article/details/106136196)

### 远程桌面服务:x11vnc


1. 安装x11vnc
```
sudo pacman -S x11vnc
```
2. 生成密码
```
x11vnc -storepasswd
```
3. 开启服务
```
x11vnc -auth guess -once -loop -noxdamage -repeat -rfbauth /home/USERNAME/.vnc/passwd -rfbport 5900 -shared
```

或
```
x11vnc -forever -shared -rfbauth ~/.vnc/passwd
```

> 注意：/home/USERNAME/.vnc/passwd 中的USERNAME需要换成你自己的用户名。
> 更多参数说明，请参考http://www.karlrunge.com/x11vnc/x11vnc_opts.html

4. 设为开机启动
```
sudo nano /lib/systemd/system/x11vnc.service
```
在打开的页面中插入以下代码
```
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -once -loop -noxdamage -repeat -rfbauth /home/USERNAME/.vnc/passwd -rfbport 5900 -shared

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
```

此时就可以在vncviewer中登陆了。
使用 ip:port 和 梗菜设置的密码登陆就好了。

> 注意: windows下需要下载VNC软件客户端，推荐用TightVNC[下载地址](https://www.tightvnc.com/download.php) [vncserver官方说明请查看](https://help.ubuntu.com/community/VNC/Servers) 通过VNCVIEW工具链接远程桌面，且输入上面设置的密码就可以看到桌面

5. 修改vnc viewer屏幕分辨率
使用man命令获得关于geometry参数的描述
```
[root@secdb ~]# man vncserver
……
       -geometry widthxheight
              Specify the size of the desktop to be created. Default is 1024x768.
……
```

可见，默认的分辨率是1024x768，我们可以使用这个参数对分辨率进行调整。
例如，我们需要将分辨率调整到800x600
```
[root@secdb ~]# vncserver -geometry 800x600

New 'secdb:5 (root)' desktop is secdb:5

Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/secdb:5.log
```
此时使用“192.168.23.102:5”登录VNC便会得到一个800x600的操作窗口。
其他分辨率调整请自行尝试。

6. 补充； 配置虚拟分辨率
服务器如果没有外接外接显示器，x-session不能从外部获取分辨率，需要在xorg.conf中设置虚拟分辨率。
> 参考：http://askubuntu.com/questions/100604/set-desktop-resolution-for-standard-11-10-vnc-server


7. SSH端口转发加密渠道安全地访问远程X服务

为了安全地使用x11vnc，您首先需要安装并且配置好SSH。
在启动x11vnc的时候，指定命令行选项“-localhost”，这将导致VNC服务只能绑定到本地网络界面。此时从外界直接连入的连接将被拒绝。
当您需要从另一台电脑上访问这个VNC服务的时候，首先用SSH登录到运行VNC的主机，将VNC服务监听的端口转发到您的本地主机。以下的例子中假设运行VNC的主机名为"foo"，VNC监听5900端口上：
ssh foo -L 5900:localhost:5900
SSH连接建立以后，打开VNC客户端程序，但是不要让它连接到foo的5900端口，而是连接到本机（localhost）的5900端口。
这样，您就可以通过加密渠道安全地访问远程X服务了。

> 参考资料
- [使用x11vnc作为vncserver端](https://www.cnblogs.com/qiaoyanlin/p/6914530.html)
- [Ubuntu远程SSH及x11vnc远程桌面连接](https://blog.csdn.net/ywueoei/article/details/79952727)
- [X11vnc (简体中文)](https://wiki.archlinux.org/index.php?title=X11vnc_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)&mobileaction=toggle_view_mobile)

### 远程桌面连接：Remmina 

---
```
	安装（连接Windows）
	sudo pacman -S remmina freerdp libvncserver telepathy-glib nxproxy xorg-server-xephyr gnome-keyring spice-gtk3
```

### 网络扫描和嗅探工具：nmap

---
```
	安装
	sudo pacman -S nmap

	使用

		nmap ip/域名                                   扫描出其对外开放服务

		nmap -p 端口1,端口2, ip                        扫描指定ip的某端口是否开放

		nmap -iL 文件路径                              扫描文档中所有主机（一行一个IP）
		
		nmap ip/24   如：nmap 192.168.43.0/24          扫描一个网段主机存活情况
		
		nmap -sP ip/24   如：nmap -sP 192.168.43.0/24  扫描这个网段的主机

		nmap -sn ip/24  如: nmap -sn 192.168.43.0/24   ping探测扫描主机（不进行端口扫描）


```

### 终端: st

+ [st软件开源官网地址](http://st.suckless.org/)

+ 鼠标选画终端脚本
```
##########################################################################
# File Name: mouse-create-st.sh  鼠标选画终端窗口脚本
# Author: Dimerbone
# mail: 15857404828@163.com
# Created Time: 2021年02月11日 星期四
# 依赖
# sudo pacman -S slop
# > slop（Select Operation）是一个应用程序，它从用户那里查询选择并将区域打印到stdout
#########################################################################
#!/bin/zsh
#!/usr/bin/env bash
# 获取鼠标选择区域标准输入值
#slop -f "%x %y %w %h" -b 1 -t 0 -q >> debug.log
read -r X Y W H < <(slop -f "%x %y %w %h" -b 1 -t 0 -q)
# Width and Height in px need to be converted to columns/rows
# To get these magic values, make a fullscreen st, and divide your screen width by ${tput cols}, height by ${tput lines} 
(( W /= 8 ))
(( H /= 16 ))
g=${W}x${H}+${X}+${Y}
# 设置i3桌面模式为悬浮模式
# 在i3设置 类别为mouse-st的窗口为悬浮 for_window [class="mouse-st"] floating enable
# 设置bspcwm桌面模式为悬浮模式
#bspcwm rule -a st-256color -o state=floating
if [ "$1" == "perl6" ]; then
    st -c "mouse-st" -g $g -e perl6 
else
    st -c "mouse-st" -g $g  
fi
```
> [鼠标选画终端脚本 github地址](https://github.com/f0x52/dots/blob/master/bin/bin/select_st)

> 参考资料
- [装b新技能，执手(鼠)画终端——archlinux,i3演示第二季](https://www.bilibili.com/video/BV1x7411c7tG?from=search&seid=7919922028367493020)

### 编辑器: vscode

```
yay -S visual-studio-code-bin
```

### 终端（shell）软件：Zsh

---
```
    sudo pacman -S zsh

    配置：
       echo $SHELL    //  查看一下自己目前使用的终端是什么
       zsh  // 检查 Zsh 是否被正确得安装

    将Zsh作为你的默认终端
        列出已安装的shell：
            $ chsh -l
        设置shell为zsh：
            $chsh -s /bin/zsh
            
        设置默认shell：
            $ chsh -s <完整路径到shell>
    卸载
        在卸载 zsh 之前请先更换默认终端。

        警告: 如果不遵循下面的步骤可能会导致用户无法访问任何终端
        运行下面的命令：
        $ chsh -s /bin/bash user

        也可以以 root 身份修改 /etc/passwd 文件，来批量更改用户的默认终端。
            警告: 强烈建议使用 vipw 来修改 /etc/passwd，因为它可以帮助你消灭格式错误
        例如将下面的配置中的 /bin/zsh
        username:x:1000:1000:Full Name,,,:/home/username:/bin/zsh
        改成 /bin/bash
        username:x:1000:1000:Full Name,,,:/home/username:/bin/bash
```

+ 配置zsh

```
wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh

或

git clone https://github.com/robbyrussell/oh-my-zsh.git ～/.oh-my-zsh


```

+ 扩展

```


cdf(){
	cd $( fd -p '/' -t d |  fzf --height=60% --preview 'ls -alh {}')
}

vimf() {
	vim $( fzf --height=60% --preview 'cat {}')
}


## 终端命令行查英语单词
v2(){
	declare q="$*";
	#curl --user-agent curl "https://v2en.co/${q// /%20}";
	curl --user-agent curl "https://d.supjohn.com/${q// /%20}";

}
## v2-sh 可以直接进入交互模式，不用重复输入 v2 前缀
v2-sh(){
	while echo -n "v2en>";
	read -r input;
	[[ -n "$input" ]]
	do v2 "$input";
	done
}
```

+ 使用
```

关闭每天会在第一次打开时进行自动更新检测
vim ~/.zshrc
查找DISABLE_AUTO_UPDATE，将其注释打开，使其生效
DISABLE_AUTO_UPDATE="true"

手动更新命令
    upgrade_oh_my_zsh 
    或
    omz update
当然也可以通过下面的方式来进行检查更新：
zsh ~/.oh-my-zsh/tools/check_for_upgrade.sh

设置主题
    omz theme set <主题名>


```

> 参考资料
- [Ubuntu | 安装oh-my-zsh](https://www.jianshu.com/p/ba782b57ae96)
- [oh my zsh 升级命令](https://blog.csdn.net/u011675334/article/details/109149782)
- [oh-my-zsh国内镜像安装和更新方法](https://blog.csdn.net/qq_39530754/article/details/104714976)

### 容器: docker、docker-compose

---
```
	安装docker
		sudo pacmam -S docker

	安装docker-compose
		安装方法：直接使用二进制文件  
		安装 Docker Compose 可以通过下面命令自动下载适应版本的 Compose，并为安装脚本添加执行权限  

		示例1：  
		sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose  
		sudo chmod +x /usr/local/bin/docker-compose  

```


### 翻页时钟：gluqlo

+ 安装gluqlo
```
git clone https://github.com/alexanderk23/gluqlo
cd gluqlo
sudo pacman -S sdl sdl_gfx sdl_ttf
sudo make 
sudo make install
sudo cp ./gluqlo /usr/local/bin/
sudo cp ./gluqlo.desktop /usr/local/applications/
```

+ 配置
```
如果您想将Gluqlo用作屏幕保护程序，则可能需要删除gnome-screensaver（现在什么都不做）并安装XScreensaver。
不要忘记将gluqlo添加到〜/ .xscreensaver配置文件中（在programs:部分中）：

gluqlo -root \n\
```

> 参考资料
- [Gluqlo: Fliqlo for Linux Github项目地址](https://github.com/alexanderk23/gluqlo)

### XScreenSaver

+ 安装
```
sudo pacman -S xscreensaver

yay -S xscreensaver-arch-logo
AUR 包，可以获得有 Arch Linux 标志的外观
```

+ 使用
```
如果要立即触发 xscreensaver，如果它正在运行，并锁定了屏幕，请执行以下命令：
xscreensaver-command -lock
```
> 参考资料
- [XScreenSaver (简体中文)](https://wiki.archlinux.org/index.php/XScreenSaver_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

### 纯命令行看图片方法

---
```
	方法一：img2txt
		img2txt filename.png
		注：安装img2txt命令： sudo pacman -S img2txt  

	~~方法二：fbi
		fbi filename.png
        注： 安装fbi命令： sudo pacman -S fbi
        
```

### 终端纯命令行看视频方法

```
	有两个库支持该特性：aa 和caca
	使用libaa，你只能在黑白ASCII中观看电影。而libcaca支持色彩。然而libaa 支持更广泛。
	命令 maplayer -vo aa 文件名

	注：如果在播放时发现视频只有一部分被显示，可以调整纵横比，可以加上-aspect选项，后面加上合适的比例，如3：1,4：1等，示例：mplayer -vo aa -aspect 4：1 

	使用libcaca，命令为 mplayer -vo caca 文件名 

	在播放时，p 或Space -暂停/继续播放。q或ESC --退出MPlayer
```


### AUR助手：aurman

---
```
    安装：
        git clone https://aur.archlinux.org/aurman.git
        cd aurman
		makepkg -si

    使用：
        aurman -Ss <package-name>                         // 用名字搜索
        aurman -S  <package-name>                         // 安装

    问题处理：

    报错：gpg
        gpg --recv-keys xxxxxx
        gpg --lsign-key xxxxxx
        gpg --finger    xxxxxx
        注： 不用sudo

	报错：==> ERROR: Cannot find the strip binary required for object file stripping
		pacman -Sy base-devel

	
```


```
 
    签名问题
    
	安装pacman-key —init

	解决签名问题

	1、屏蔽签名
		修改/etc/pacman.conf，将所有原有的SigLevel=××××××注释掉，添加SigLevel = Never即可。

	2、安装archlinux签名
		sudo pacman -Syy
		sudo pacman -S archlinux-keyring 

	3、安装pgp签名
		pacman -Syu haveged
		systemctl start haveged
		systemctl enable haveged
		rm -rf /etc/pacman.d/gnupg

	4：重新签名
		pacman-key —init
		pacman-key —populate manjaro
		pacman-key —populate archlinux

```

```
错误：一个或多个文件没有通过有效性检查

```


> 参考资料

+ [makepkg遇到问题ERROR: Cannot find the strip binary required for object file stripping.](https://www.jianshu.com/p/e053d0ac1212)


### AUR助手：Yaourt

---
```
	简单安装Yaourt的方式是添加Yaourt源至您的 /etc/pacman.conf:
	[archlinuxcn]
	#The Chinese Arch Linux communities packages.
	SigLevel = Optional TrustAll
	Server   = http://repo.archlinuxcn.org/$arch

	同步并安装：
		sudo pacman -Syu yaourt
		即可使用 yaourt 安装程序，例如写博客使用的 markdown 编辑器 remarkable ：
		sudo yaourt remarkable


	LANG=en_US sudo pacman -Sy archlinux-keyring 
```

> 参考资料

+ [Index» Pacman & Package Upgrade Issues» [solved] Cannot install faulty signed packages](https://bbs.archlinux.org/viewtopic.php?pid=1732412)
+ [关于ARCHLINUX在TERMUX中使用AUR的解决方案](http://www.freesion.com/article/546736653/)
+ [[解决]Win10 WSL系统下编译buildroot报错不支持SYSV IPC，导致fakeroot无法正常工作](https://whycan.cn/t_1004.html)

### AUR助手：yay

---
```
    安装：
        git clone https://aur.archlinux.org//yay.git
        cd ./yay
        makepkg -si

    使用
        yay -Ss <package-name>                          // 用名字搜索
        yay -S <package-name>                           // 安装

```
```
解决 go get获取package时候time out超时问题

正常使用go get获取安装包时候因为一些原因，导致经常出现超时的问题，特别，go在后面的版本（go.1.13）增加了一个功能，即可以使用代理获取package。

Mac、Linux
	临时生效
		export GOPROXY=https://goproxy.io,direct"
	长久生效	
		# 设置你的 bash 环境变量
		echo "export GOPROXY=https://goproxy.io,direct" >> ~/.profile && source ~/.profile

		# 如果你的终端是 zsh，使用以下命令
		echo "export GOPROXY=https://goproxy.io,direct" >> ~/.zshrc && source ~/.zshrc


Window
	临时生效
		Windows_Powershell_22
		# 配置 GOPROXY 环境变量
		$env:GOPROXY = "https://goproxy.io,direct"
	长久生效
		1. 右键 我的电脑 -> 属性 -> 高级系统设置 -> 环境变量
		2. 在 “[你的用户名]的用户变量” 中点击 ”新建“ 按钮
		3. 在 “变量名” 输入框并新增 “GOPROXY”
		4. 在对应的 “变量值” 输入框中新增 “https://goproxy.io,direct”
		5. 最后点击 “确定” 按钮保存设

验证配置是否成功
	go env |grep PROXY 

```
> 参考资料
- [解决 go get获取package时候time out超时问题](https://blog.csdn.net/weixin_43420337/article/details/117553023)


### 百度云盘/百度网盘: baidunetdisk

```
yay -S baidunetdisk
```

> 参考资料
- [arch配置之百度网盘](https://blog.csdn.net/weixin_44451210/article/details/92384191)

### 坚果云: 

+ 方式一：

```
1. 准备构建环境

坚果云Linux客户端依赖于这些包: glib2.0-dev, gtk2.0-dev, libnautilus-extension-dev, gvfs-bin. 如果您已经安装这些软件包，请跳至下一步

如果您的系统是Archlinux可以用以下命令安装这些包：
sudo pacman -S libnautilus-extension
sudo pacman -S gvfs
sudo pacman -S libappindicator-gtk3
sudo pacman -S python2-gobject

2. 下载Nautilus插件源代码包: nutstore_linux_src_installer.tar.gz
$> wget https://www.jianguoyun.com/static/exe/installer/nutstore_linux_src_installer.tar.gz
或
$> wget https://www.jianguoyun.com/static/exe/installer/nutstore_linux_dist_x64.tar.gz

3. 解压缩，编译和安装Nautilus插件
$> tar zxf nutstore_linux_src_installer.tar.gz
$> cd nutstore_linux_src_installer && ./configure && make
$> sudo make install

4. 重启Nautilus
$> nautilus -q

5. 运行以下命令，自动下载和安装坚果云其他二进制组件
$> ./runtime_bootstrap
```

+ 方式一:
```
sudo pacman -S gvfs
sudo pacman -S libappindicator-gtk3
sudo pacman -S python2-gobject
sudo pacman -S wget
wget https://www.jianguoyun.com/static/exe/installer/nutstore_linux_src_installer.tar.gz
tar zxf nutstore_linux_src_installer.tar.gz
cd nutstore_linux_src_installer && ./configure && make
sudo make install
./runtime_bootstrap
```


> 参考资料
- [](https://www.jianguoyun.com/s/downloads/linux)
- [坚果云客户端 在arch KDE上安装](https://blog.csdn.net/usenet506/article/details/94226200)

### 终端的任务监控中心： htop

---
```
    sudo pacman -S htop

```
### 终端的任务监控中心： gtop
---
```
    sudo pacman -S gtop
```
### 电池管理软件

---
```
	安装
	sudo pacman -S acpi

    acpi // 电池信息查看

```

### 安装jdk环境  

---  
```  

    1. 从官网下载jdk1.8的tar包  
        注：本次使用包为 jdk-8u211-linux-x64.tar.gz  

    2. 将下载的jdk的tar包解压到/usr/local/  
        tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/  

        //x : 从 tar 包中把文件提取出来  
        //z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压  
        //v : 显示详细信息  
        //f xxx.tar.gz :  指定被处理的文件是 xxx.tar.gz,这几个连用的时候f必须放在最后  
        //-C：指定需要解压到的目录。  

    3. 给解压后的文件创建一个软链接便于以后的版本更替  
        cd /usr/local/  
        ln -s jdk1.8.0_211/ java  

    4. 在/etc/profile中最下面添加java环境配置:  
        vim /etc/profile  

        export JAVA_HOME=/usr/local/java  
        export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib  
        export PATH=$PATH:$JAVA_HOME/bin  

    5、启动/etc/profile文件中的配置:  
        source /etc/profile  

        注意:普通用户登陆系统然后在shell中切换到root启动配文件后再切换回普通用户后配置文件对于普通用户并没有起效果.  
        此时可以注销或者在普通用户环境下再启动一次配置文件.  

    5.简单的java程序测试，查看版本号  
        java -version  

```  

> 参考资料  
+ [Linux使用tar包安装jdk1.8](https://blog.csdn.net/liuchonghua/article/details/82698265)  


### IDE：Idea

---
```
    1. 从官网下载idea的tar包  
        注：本次使用包为

    2. 将下载的idea的tar包解压到/usr/local/  
        tar -zxvf idea-IU-xx.tar.gz  -C /usr/local/  

        //x : 从 tar 包中把文件提取出来  
        //z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压  
        //v : 显示详细信息  
        //f xxx.tar.gz :  指定被处理的文件是 xxx.tar.gz,这几个连用的时候f必须放在最后  
        //-C：指定需要解压到的目录。  

	3. 启动idea，进入解压的idea的文件夹的下的./bin/idea.sh

	4. 加入应用列表，以便dmenu快速打开
		查看执行 echo $PATH 查看环境变量
		建立执行软链接
		sudo ln -s /usr/local/idea-IU-201.6668.113/bin/idea.sh /usr/bin/idea

	5. 创建桌面快捷启动方式
		在/usr/share/applications/目录下创建一个idea.desktop文件
			[Desktop Entry]
			Type=Application
			Name=idea
			Comment=IntelliJIDEA
			Terminal=false
			Icon=/usr/local/idea-IU-201.6668.113/bin/idea.png
			Exec=/usr/local/idea-IU-201.6668.113/bin/idea.sh
       并赋予执行权限 chmod +x /usr/share/applications/idea.desktop



```


```
安装包及破解idea2019.3文件
	下载链接：https://pan.baidu.com/s/1-kE2uWEsf0Ko6vKXlLJTEw 提取码：34dq。



安装教程：
	第一步：正常的安装idea，一直下一步到finish
	第二步：将刚才的破解文件 （jetbrains-agent.jar） 放入安装的目录的bin目录下
	可以参考我的 /usr/local/ideaIU-2019.3.4xxxxx/bin

	第三步：打开idea，并选择免费试用
	第四步：点击最上方的Help->Edit Custom VM Options
	如果提示要创建文件，点击“yes”，没有的话忽略，这时候会打开一个文件。

	第五步：在文件的最下面添加上一句话
	-javaagent:自己idea的安装目录\jetbrains-agent.jar
	即：-javaagent:/usr/local/ideaIU-2019.3.4xxxxx/bin/jetbrains-agent.jar

	第六步：先重启idea（必须），然后点击Help->Register...

	第七步：
	法一：
		选择最后一个License server
		输入：http://fls.jetbrains-agent.com
		或者：http://jetbrains-license-server
		然后点击Activate

	法二：
	选择Activation code方式，拷贝如下代码（有点长别复制错了）：
	A82DEE284F-eyJsaWNlbnNlSWQiOiJBODJERUUyODRGIiwibGljZW5zZWVOYW1lIjoiaHR0cHM6Ly96aGlsZS5pbyIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiJVbmxpbWl0ZWQgbGljZW5zZSB0aWxsIGVuZCBvZiB0aGUgY2VudHVyeS4iLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlMwIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiV1MiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJSRCIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlJDIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREMiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJEQiIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlJNIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiRE0iLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJBQyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkRQTiIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkdPIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUFMiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJDTCIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlBDIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlNVIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In1dLCJoYXNoIjoiODkwNzA3MC8wIiwiZ3JhY2VQZXJpb2REYXlzIjowLCJhdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJpc0F1dG9Qcm9sb25nYXRlZCI6ZmFsc2V9-5epo90Xs7KIIBb8ckoxnB/AZQ8Ev7rFrNqwFhBAsQYsQyhvqf1FcYdmlecFWJBHSWZU9b41kvsN4bwAHT5PiznOTmfvGv1MuOzMO0VOXZlc+edepemgpt+t3GUHvfGtzWFYeKeyCk+CLA9BqUzHRTgl2uBoIMNqh5izlDmejIwUHLl39QOyzHiTYNehnVN7GW5+QUeimTr/koVUgK8xofu59Tv8rcdiwIXwTo71LcU2z2P+T3R81fwKkt34evy7kRch4NIQUQUno//Pl3V0rInm3B2oFq9YBygPUdBUbdH/KHROyohZRD8SaZJO6kUT0BNvtDPKF4mCT1saWM38jkw==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5ndaik1GD0nyTdqkZgURQZGW+RGxCdBITPXIwpjhhaD0SXGa4XSZBEBoiPdY6XV6pOfUJeyfi9dXsY4MmT0D+sKoST3rSw96xaf9FXPvOjn4prMTdj3Ji3CyQrGWeQU2nzYqFrp1QYNLAbaViHRKuJrYHI6GCvqCbJe0LQ8qqUiVMA9wG/PQwScpNmTF9Kp2Iej+Z5OUxF33zzm+vg/nYV31HLF7fJUAplI/1nM+ZG8K+AXWgYKChtknl3sW9PCQa3a3imPL9GVToUNxc0wcuTil8mqveWcSQCHYxsIaUajWLpFzoO2AhK4mfYBSStAqEjoXRTuj17mo8Q6M2SHOcwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQBonMu8oa3vmNAa4RQP8gPGlX3SQaA3WCRUAj6Zrlk8AesKV1YSkh5D2l+yUk6njysgzfr1bIR5xF8eup5xXc4/G7NtVYRSMvrd6rfQcHOyK5UFJLm+8utmyMIDrZOzLQuTsT8NxFpbCVCfV5wNRu4rChrCuArYVGaKbmp9ymkw1PU6+HoO5i2wU3ikTmRv8IRjrlSStyNzXpnPTwt7bja19ousk56r40SmlmC04GdDHErr0ei2UbjUua5kw71Qn9g02tL9fERI2sSRjQrvPbn9INwRWl5+k05mlKekbtbu2ev2woJFZK4WEXAd/GaAdeZZdumv8T2idDFL7cAirJwcrbfpawPeXr52oKTPnXfi0l5+g9Gnt/wfiXCrPElX6ycTR6iL3GC2VR4jTz6YatT4Ntz59/THOT7NJQhr6AyLkhhJCdkzE2cob/KouVp4ivV7Q3Fc6HX7eepHAAF/DpxwgOrg9smX6coXLgfp0b1RU2u/tUNID04rpNxTMueTtrT8WSskqvaJd3RH8r7cnRj6Y2hltkja82HlpDURDxDTRvv+krbwMr26SB/40BjpMUrDRCeKuiBahC0DCoU/4+ze1l94wVUhdkCfL0GpJrMSCDEK+XEurU18Hb7WT+ThXbkdl6VpFdHsRvqAnhR2g4b+Qzgidmuky5NUZVfEaZqV/g==



	第八步：over，破解完成
```


```
安装包及破解idea2020.1文件
	idae2020.1最新激活教程
	链接：https://pan.baidu.com/s/19s5tYmpRF5GOkObMMSN2qA 提取码：r9x4
	如失效请留言。

安装教程：
	第一步：正常的安装idea，一直下一步到finish
	第二步：将刚才的破解文件 （idea2020.1破解包/jar/jetbrains-agent.jar） 放入安装的目录的bin目录下
	可以参考我的 /usr/local/idea-IU-201.6668.113/bin

	第三步：打开idea，并选择免费试用
	第四步：点击最上方的Help->Edit Custom VM Options
	如果提示要创建文件，点击“yes”，没有的话忽略，这时候会打开一个文件。

	第五步：在文件的最下面添加上一句话
	-javaagent:自己idea的安装目录\jetbrains-agent.jar
	即：-javaagent:/usr/local/idea-IU-201.6668.113/bin/jetbrains-agent.jar

	第六步：先重启idea（必须），然后点击Help->Register...

	重启动后会变为
	-javaagent:/home/用户名/.jetbrains/jetbrains-agent-v3.0.0.jar

	第七步：
    选择最后一个License server
    http://fls.jetbrains-agent.com

	选择Activation code方式，拷贝如下代码（有点长别复制错了）：
	3AGXEJXFK9-eyJsaWNlbnNlSWQiOiIzQUdYRUpYRks5IiwibGljZW5zZWVOYW1lIjoiaHR0cHM6Ly96aGlsZS5pbyIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkFDIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkRQTiIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJQUyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJHTyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJETSIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJDTCIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJSUzAiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkQiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUEMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUk0iLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiV1MiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREIiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlNVIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9XSwiaGFzaCI6IjEyNzk2ODc3LzAiLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0=-WGTHs6XpDhr+uumvbwQPOdlxWnQwgnGaL4eRnlpGKApEEkJyYvNEuPWBSrQkPmVpim/8Sab6HV04Dw3IzkJT0yTc29sPEXBf69+7y6Jv718FaJu4MWfsAk/ZGtNIUOczUQ0iGKKnSSsfQ/3UoMv0q/yJcfvj+me5Zd/gfaisCCMUaGjB/lWIPpEPzblDtVJbRexB1MALrLCEoDv3ujcPAZ7xWb54DiZwjYhQvQ+CvpNNF2jeTku7lbm5v+BoDsdeRq7YBt9ANLUKPr2DahcaZ4gctpHZXhG96IyKx232jYq9jQrFDbQMtVr3E+GsCekMEWSD//dLT+HuZdc1sAIYrw==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5ndaik1GD0nyTdqkZgURQZGW+RGxCdBITPXIwpjhhaD0SXGa4XSZBEBoiPdY6XV6pOfUJeyfi9dXsY4MmT0D+sKoST3rSw96xaf9FXPvOjn4prMTdj3Ji3CyQrGWeQU2nzYqFrp1QYNLAbaViHRKuJrYHI6GCvqCbJe0LQ8qqUiVMA9wG/PQwScpNmTF9Kp2Iej+Z5OUxF33zzm+vg/nYV31HLF7fJUAplI/1nM+ZG8K+AXWgYKChtknl3sW9PCQa3a3imPL9GVToUNxc0wcuTil8mqveWcSQCHYxsIaUajWLpFzoO2AhK4mfYBSStAqEjoXRTuj17mo8Q6M2SHOcwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQBonMu8oa3vmNAa4RQP8gPGlX3SQaA3WCRUAj6Zrlk8AesKV1YSkh5D2l+yUk6njysgzfr1bIR5xF8eup5xXc4/G7NtVYRSMvrd6rfQcHOyK5UFJLm+8utmyMIDrZOzLQuTsT8NxFpbCVCfV5wNRu4rChrCuArYVGaKbmp9ymkw1PU6+HoO5i2wU3ikTmRv8IRjrlSStyNzXpnPTwt7bja19ousk56r40SmlmC04GdDHErr0ei2UbjUua5kw71Qn9g02tL9fERI2sSRjQrvPbn9INwRWl5+k05mlKekbtbu2ev2woJFZK4WEXAd/GaAdeZZdumv8T2idDFL7cAirJwcrbfpawPeXr52oKTPnXfi0l5+g9Gnt/wfiXCrPElX6ycTR6iL3GC2VR4jTz6YatT4Ntz59/THOT7NJQhr6AyLkhhJCdkzE2cob/KouVp4ivV7Q3Fc6HX7eepHAAF/DpxwgOrg9smX6coXLgfp0b1RU2u/tUNID04rpNxTMueTtrT8WSskqvaJd3RH8r7cnRj6Y2hltkja82HlpDURDxDTRvv+krbwMr26SB/40BjpMUrDRCeKuiBahC0DCoU/4+ze1l94wVUhdkCfL0GpJrMSCDEK+XEurU18Hb7WT+ThXbkdl6VpFdHsRvqAnhR2g4b+Qzgidmuky5NUZVfEaZqV/g==

	第八步：over，破解完成

```

```
不破解重复使用30天免费
    Linux下删除 ～/.config/JetBrains/IntelliJIdea2020.1 
    Windows下删除 C:\Users\用户名\AppData\Roaming\JetBrains

```

> 参考资料

+ [IntelliJ IDEA安装破解教程（附安装包2019.3 版+破解文件+激活方法)](https://www.cnblogs.com/chaogu94/p/12175819.html)
+ [2019版 IntelliJ IDEA破解，2020/3/11更新](https://www.jianshu.com/p/c6192e540d45)

+ [idea 2020年最新破解补丁（亲测有效）](http://errornoerror.com/question/1585959663051139/) 


### dwm下安装idea遇到问题：
	
1. 桌面下 打开时只有主题的纯色
```
解决办法：
	sudo pacman -S wmname
	添加下面的代码到~/.xinitrc

	export _JAVA_AWT_WM_NONREPARENTING=1 
	export AWT_TOOLKIT=MToolkit 
	wmname LG3D

之后reboot或者source ~/.xinitrc
```

> 参考资料

+ [Linux系统安装 IntelliJ IDEA](https://www.cnblogs.com/418836844qqcom/p/10722197.html)

+ [求问:archlinux下使用dwm窗口,运行idea的时候,设置页面打开项目过程都是正常的,项目打开之后就整个窗口就变成了纯主题色,像是黑屏那种](https://blog.csdn.net/weixin_44513641/article/details/104738842)
+ [idea-dwm](https://blog.csdn.net/u010563350/article/details/104948256)

+ [在linux中添加应用程序到applications列表](https://blog.csdn.net/huahuajjh/article/details/63263485)

## HTTP请求调试器:Postman

+ 安装Postman shell脚本install-postman.sh：
```
#!/bin/bash
cd /tmp || exit
echo "Downloading Postman ..."
wget -q https://dl.pstmn.io/download/latest/linux?arch=64 -O postman.tar.gz
tar -xzf postman.tar.gz
rm postman.tar.gz

echo "Installing to opt..."
if [ -d "/opt/Postman" ];then
    sudo rm -rf /opt/Postman
	fi
	sudo mv Postman /opt/Postman

	echo "Creating symbolic link..."
	if [ -L "/usr/bin/postman" ];then
	    sudo rm -f /usr/bin/postman
		fi
		sudo ln -s /opt/Postman/Postman /usr/bin/postman

		echo "Installation completed successfully."
```

+ Postman.desktop文件：
```
[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=postman
Icon=/opt/Postman/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
```
- [安装Postman shell脚本（包括.desktop文件）](https://majing.io/posts/10000009081174)

## 微信开发者工具：wecha-web-devtools

+ 运行准备:
```
	GUI环境
	需要安装wine
		注： 默认配置的包管理器搜不到wine,需要修改配置
			[Official repositories (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/Official_repositories_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)
		启用multilib仓库
		multilib 仓库位于仓库镜像的 .../multilib/os/ 目录。
		包含64位系统中需要的32位软件和库，例如： wine 等。

		启用
		想使用 multilib 仓库，编辑 /etc/pacman.conf，取消下面内容的注释：

		[multilib]
		Include = /etc/pacman.d/mirrorlist
		更新软件包列表并升级系统 pacman -Syu.

	sudo pacman -S  wine
```

+ 方式一：
```
	版本1
	aurman -S wechat-devtool
	或
	#yay -S wecha-web-devtool
	版本2
    yay -S wecha-web-devtools
```

+ 方式三：

```

	git clone https://github.com/cytle/wechat_web_devtools.git
	cd wechat_web_devtools
	# 自动下载最新 `nw.js` , 同时部署目录 `~/.config/wechat_web_devtools/`
	./bin/wxdt install

	cd ..
	mv ./wechat_web_devtools  /usr/local/bin/

	cd /usr/local/bin/wechat_web_devtools/bin

	建立一个启动脚本start_wxdt.sh
	#!/bin/bash
	/usr/local/bin/wechat_web_devtools/bin/wxdt 

	sudo chmod +x ./start_wxdt.sh
	sudo ln -s /usr/local/bin/wechat_web_devtools/bin/wxdt /usr/local/bin/wechat_web_devtools
	sudo chmod +x /usr/local/bin/wechat_web_devtools

```

- [Linux微信web开发者工具 - TBHacker - 博客园](https://www.cnblogs.com/jiqing9006/p/8992038.html)

### 小狼豪输入法/中州韻輸入法引擎: rime

+ ArchLinux下安裝
```
	方式一： fcitx框架下
		sudo pacman -S fcitx fcitx-im fcitx-configtool fcitx-rime
		sudo pacman -S ttf-dejavu wqy-microhei
		添加对 gtk、qt的支持vim  /etc/profile
			#export LC_ALL=“zh_CN.UTF-8”
			export GTK_IM_MODULE=fcitx
			export QT_IM_MODULE=fcitx
			export XMODIFIERS="@im=fcitx"

	基本配置
		以下提到的配置文件皆放置或创建在~/.config/fcitx/rime文件夹下。
		在~/.config/fcitx/rime/中新建default.custom.yaml
	
	扩展词库
		RIME的默认词库并没有多大，因为作者希望用户在与 RIME 磨合的过程中自己积累用户词库，确实自己养起来的词库更顺手，但养词库的过程多少还是有点痛苦的，好在RIME支持扩展词库。
		这里使用的是 xiaoTaoist 制作的词库扩展包
		GitHub 地址  下载地址
		使用方法
		将解压出来的所以文件复制到~/.config/fcitx/rime文件夹下，将luna_pinyin.custom.yaml重命名为luna_pinyin_simp.custom.yaml
		这个扩展词库的luna_pinyin.custom.yaml中也包含了模糊拼音的功能，按注释开启即可。

		右下角图标->重新部署，检查是否运行成功

```

```
	快速打开选择 繁体简体
	rime默认是繁体。
	随意输入一段文字，按住Ctrl + `,选择第4个。
```

> 參考資料

- [Rime输入法配置 - 简书](https://www.jianshu.com/p/3e967c0f40b7)
- [linux中rime输入法安装使用小结 - 简书](https://www.jianshu.com/p/58ea12e8886d)

### 输入框架：fcitx5

```
sudo pacman -S fcitx5 fcitx5-configtool  

环境变量配置
～/.pam_environment

INPUT_METHOD  DEFAULT=fcitx5
GTK_IM_MODULE DEFAULT=fcitx5
QT_IM_MODULE  DEFAULT=fcitx5
XMODIFERS     DEFAULT=\@im=fcitx5

其他

sudo pacman -S fcitx5-rime fcitx5-im fcitx5-chinese-addons  fcitx5-chewing

sudo fcitx5-qt fcitx5-gtk

词库
yay -S fcitx5-pinyin-zhwiki
yay -S fcitx5-pinyin-moegirl

yay -S fcitx5-pinyin-zhwiki-rime
yay -S fcitx5-pinyin-moegirl-rime

```


### 解deb解包工具:debtap


+ 首先查看电脑上是否安装过
	```
	sudo pacman -Q debtap
	```
+ 安装yay工具，记得配置arch
	```
	sudo pacman -S yay
	```
+ 安装解包打包工具debtap
	```
	yay -S debtap
	```
+ 升级debtap
	```
	sudo debtap -u
	```
+ 使用debtap工具进行解包
	```
	sudo debtap  xxxx.deb
	```
+ 安装
	```
	sudo pacman -U x.tar.xz
	```

> ps
	```
	ar -x 包名.deb     可解压出deb里面的文件
	解压后文件件
	./
	|-- data.tar.xxx         数据包	包含实际安装的程序数据，文件名可能为“data.tar.zx 等”
	|    |-- /usr
	|    |-- /opt
	|    |-- ...
	|    \..
	|-- control.tar.gz       安装信息及控制包	包含deb的安装说明，标识，脚本等，文件名为“control.tar.gz”
	|-- debian-binary        二进制数据	包含文件头等信息，需要特殊软件才能查看
	\..
	```

	```
	使用debtap解决包后
	```

> 参考资料
- [Arch/Manjaro安装deb安装包](https://www.jianshu.com/p/7acbffa17c28)
- [Ubuntu中deb包详解及打包教程](https://blog.csdn.net/qq_16149777/article/details/86672286)

### PKGBUILD archlinux 打包

```
编写 PKGBUILD
在最开头，复制一份原型： /usr/share/pacman/PKGBUILD.proto（同目录下也有其他特别类型的原型），之后就从这个文件开始编写啦。

先读完文件开头那段注释，然后删掉它～

Maintainer
最开头一行注释是维护者的信息，按照它提供的格式填写上有效的信息即可。

pkgname
软件包的名字。只能用 小写字母、数字和@ . _ + - 这些字符，且不允许用.或者-作开头。

另外不要和 AUR 甚至是官方仓库里面的软件包重名了(´・ω・｀)

pkgver
软件包的版本，就是你打包的那个软件的版本。可以使用数字和小数点，以及其它字符。进一步的规则可参考：VCS package guidelines - ArchWiki

pkgrel
软件包发行号，一般设为 1，如果你因为某些原因给同版本号的软件进行反复打包，那么每次打包的时候 pkgrel 就应该在原基础上递增一个数字，而在打包新的版本的时候，应该重新设为 1。

epoch
强行干涉包的新旧关系，拥有更大的 epoch 值的包会被认做更新的包（此时无视版本号），可以用在如版本号风格改变等需要的时候。默认值为 0，取值为正整数。一般不会用到。

pkgdesc
软件包的描述信息，最好一句话，且不包含软件的名字。

arch
表示支持的 Arch Linux 的架构，比如 i686、x86_64，如果包与平台无关的话就填 any。

url
与软件包相关的链接，一般是项目首页什么的。

license
软件发布协议，如果是常见的 GPL 的话可以对照下面填写：

(L)GPL - (L)GPLv2 及更新版本。
(L)GPL2 - 仅 (L)GPL2
(L)GPL3 - (L)GPL3 及更新版本
depends
这是非常重要的一项，需要正确填写上软件的依赖。

对于直接发布可执行程序的话，可以通过 ldd 来看程序连接了哪些库文件，结合搜索判断出具体依赖是什么软件包。你可以用谷歌在 https://www.archlinux.org 上搜索具体库的文件名，一般都能够找到对应的软件包。

如果你已经用 makepkg 打出了 .tar.xz 的包，也可以用 Namcap 来检查依赖是否存在问题，它会提供一些有用的信息帮助修正依赖。对于他的输出含义可以直接参考 ArchWiki。

多测试多测试，确保依赖真的没问题。

source
构建软件包需要的文件。可以是一个本地文件，也可以是一个远程文件。 makepkg 会在构建包的时候自动下载填写的远程文件，并且会自动解包压缩文件。

md5sums
对应的 source 里面文件的 md5 校验码。

package()
在构架包的时候执行的函数。你需要把安装软件对应的操作写在这里。函数会在一个 fakeroot 环境下执行，对应的 root 目录就是 $pkgdir，比如你有一个可执行文件名为 $pkgname 要安装到 /usr/bin 下面，对应的命令就可以类似这么写：

install -m=775 $pkgname "${pkgdir}/usr/bin"
-m 选项表示目标文件的权限，和 chmod 参数同理。

常用目录
目录	用途
/etc	系统关键配置文件，如果件有多个，应该创建合适的子目录来存放
/usr/bin	二进制文件
/usr/lib	库
/usr/include	头文件
/usr/lib/{pkg}	模块，插件等
/usr/share/doc/{pkg}	应用程序文档
/usr/share/info	GNU Info 系统文件
/usr/share/man	手册
/usr/share/{pkg}	程序数据
/var/lib/{pkg}	应用持久数据
/etc/{pkg}	{pkg}的配置文件
/opt/{pkg}	大的独立程序，例如 Java
/usr/share/applications/	Desktop Entry (.desktop) 文件
/usr/share/icons/	图标，存在该目录下对应子目录位置
不该碰的目录：

/dev
/home
/srv
/media
/mnt
/proc
/root
/selinux
/sys
/tmp
/var/tmp
构建/调试包
在 PKGBUILD 所在目录下执行 makepkg 可以构建出对应的软件包，推荐用 namcap 检测一下构建出来的包有没有更显著的问题。

然后你可以用 pacman -U 命令安装它，看看会不会发生什么奇怪的事情，以及软件是否正常等。

当然还有可能因为 PKGBUILD 没写好，直接就报错不干了，这个时候需要顺着错误信息去修正 PKGBUILD。

发布到 AUR
在 PKGBUILD 所在目录执行 makepkg --source，会生成 .src.tar.gz 源码包，这就是需要上传到 AUR 的东西，注意不要把任何二进制文件加入源码包。

在 AUR (Arch User Repository) 注册（登入）帐号。进入 Submit 页面，选择好软件包对应的分类，然后添加源码包上传即可。

即使你是要更新一个包，也只需要直接在 Submit 页面上传，包的信息 AUR 会自己处理。

如果觉得每次上传太麻烦，你可以尝试一下 aurupload 来简化发布。
```

> 参考资料
- [Arch Linux 简易打包指南](https://www.cnblogs.com/frantic1048/p/simple-arch-linux-package-guide.html)

### 搜狗输入法

---
```

    配置源
    打开/etc/pacman.conf，在末尾加上
    
    [archlinuxcn]
    SigLevel = Optional TrustAll
    Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
    //或者使用清华的镜像源
    [archlinuxcn]
    SigLevel = Optional TrustAll
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch


    安装key-ring
    $ sudo pacman -S archlinux-keyring
    更新软件仓库
    $ sudo pacman -Sy
    安装
    安装Fcitx
    由于搜狗拼音输入法依赖于Fcitx，在安装搜狗拼音输入法之前，需要先行安装Fcitx，在终端窗口下直接输入：
    
    $ sudo pacman -S fcitx
    即可完成安装，需要注意的是，仅仅安装这一项是不够的，这样在安装完成之后，Fcitx基本上是处于不可用的状态，我们还需要安装以下几个包：
    
    $ sudo pacman -S fcitx-configtool
    $ sudo pacman -S fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5
    安装搜狗拼音
    在前一步中我们已经正确的配置了源，这里直接输入：
    
    $ sudo pacman -S fcitx-sogoupinyin
    // 安装配置工具
    $ sudo pacman -S fcitx-configtool
    配置
    安装完之后我们还不可以直接使用，还需要进行一定的配置，用文本编辑器打开~/.xprofile，没有就新建，在其末尾添加以下几行：
    
    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS="@im=fcitx"
    然后注销后重新登录，或者重启后重新登录。
    
    可能的问题
    如果遇到登录之后输入法fcitx没有启动的问题，可以讲fcitx设置为自动启动，deepin桌面下右键fcitx的图片就能做到，gnome桌面可以用gnome-tweaks，也可以就简单的在.xprofile里面加一句fcitx。 
    
```

> 参考资料

+ [Archlinux安装搜狗拼音输入法](https://www.cnblogs.com/tonyc/p/8231667.html)

### 谷歌输入法

---
```
   配置源
    打开/etc/pacman.conf，在末尾加上
    
    [archlinuxcn]
    SigLevel = Optional TrustAll
    Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
    //或者使用清华的镜像源
    [archlinuxcn]
    SigLevel = Optional TrustAll
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch


    安装key-ring
    $ sudo pacman -S archlinux-keyring
    更新软件仓库
    $ sudo pacman -Sy
    安装
    安装Fcitx
    
    $ sudo pacman -S fcitx fcitx-configtool fcitx-google-pinyin
    
    编辑.xinitrc
        export GTK_IM_MODULE=fcitx
        export QT_IM_MODULE=fcitx
        export XMODIFIERS=@im=fcitx
        fcitx &
        while true; do
            xsetroot -name "Bat.$(acpi -b | awk '{print $4}') | Vol.$(amixer get Master| tail -n 1 | awk ‘{print $5}' | tr -d '[]') $(LC_ALL='C' date +'%F[%b %a] %R')"
        sleep 20
        done &
        exec dwm

        重启后用fcitx-configtool添加一下输入法

```

+ 问题： 终端或部分软件不能切换输入中文

```
解决：在/etc/profile文件中加入以下配置

# # 输入法设置
# export GTK_IM_MODULE=fcitx
# export QT_IM_MODULE=fcitx
# export XMODIFIERS=@im=fcitx
export LC_ALL=zh_CN.UTF-8
export XIM=fcitx  
export XIM_PROGRAM=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"

# > LC_ALL=zh_CN.UTF-8 是全局，包括菜单栏的文字都是中文的，如果你还想用英文的菜单，可以用以下的代码。
# export LC_CTYPE=zh_CN.UTF-8
# export XIM=fcitx
# export XIM_PROGRAM=fcitx
# export GTK_IM_MODULE=fcitx
# export QT_IM_MODULE=fcitx
# export XMODIFIERS="@im=fcitx"
# eval `dbus-launch --sh-syntax --exit-with-session`
# exec fcitx &

#
# # other
export _JAVA_AWT_WM_NONREPARENTING=1
export AWT_TOOLKIT=MToolkit
#


```

> 参考资料

+ [ArchLinux dwm的安装和配置](https://www.cnblogs.com/tanglizi/p/8457821.html)

### julia 一个面向科学计算的高性能动态高级程序设计语言

```
sudo pacman -S julia
```

### 思维导图：xmind-zen

```
yay -S xmind-zen

非破解
```

#### 安装破解Xmind-zen 2020 v10.2.1（202007272308）

第一步、下载 XMind-2020-for-Linux-amd-64bit-10.1.2-202004142327.deb
	方式一:下载deb包，使用debtap解包
		1. 安装debtap:  yay -S debtap
		2. 升级debtap:  sudo debtap -u
		3. 解包:        sudo debtap  xxxx.deb 或 sudo debtap -q xxxx.deb 
		> 你需要输入包的维护者和许可证，输入他们，然后按下回车键就可以开始转换了。包转换的过程可能依赖于你的 CPU 的速度从几秒到几分钟不等。使用 -q 略过除了编辑元数据之外的所有问题。为了略过所有的问题（不推荐），使用 -Q
		> 记:本次解包后的文件xmind-vana-10.1.2-1-x86_64.pkg.tar.zst
		4. 转换完成后，您可以使用 pacman 在 Arch 系统中安装新转换的软件包： sudo pacman -U xmind-vana-10.1.2-1-x86_64.pkg.tar.zst
		

	注：安装完成后，记得关闭软件；安装完成后，记得关闭软件；安装完成后，记得关闭软件

第二步、下载破解补丁（在压缩包中，自助解压获取解压密码）

第三步、关闭软件，使用破解补丁替换xmind安装目录下的resources\app.asar 文件即可，替换补丁时，必须关闭软件。Archlinux的安装目录是 /opt/XMind/resource

第四步、帮助-》关于查看破解信息，破解成功，而且导出的pdf和图片无任何水印，解锁所有高级功能，解锁所有高级功能，不要登录软件，否则会失效

> 参考资料
- [Xmind 2020 v10.2.1（202007272308）for windows/mac/linux安装破解教程（附激活密钥以及注册机，全网独家可用）](https://www.jianshu.com/p/07d25944b66c)
- [Arch/Manjaro安装deb安装包](https://www.jianshu.com/p/7acbffa17c28)
- [将 DEB 软件包转换成 Arch Linux 软件包](https://blog.csdn.net/nswy123/article/details/102729219)


### 在 Linux 上运行 macOS 程序 ： Darling

+ [Darling github地址](https://github.com/darlinghq/darling)

+ 安装方式
```
yay -S darling
```

> 参考资料
- [LWN: 在 Linux 上运行 macOS 程序 ](https://www.sohu.com/a/334603678_700886)

### Linux上运行Windows EXE文件： Wine for Linux

+ ArchLinux安装wine
```
使用文本编辑器打开/etc/pacman.conf，找到
#[multilib]
#Include = /etc/pacman.d/mirrorlist

将之修改为
[multilib]
Include = /etc/pacman.d/mirrorlist
而后

sudo pacman -Syu
sudo pacman -S wine

# 用于运行依赖Internet Explorer 和 .NET的程序
sudo pacman -S wine-gecko wine-mono

ArchLinux安装Deepin-Wine-TIM
sudo pacman -S deepin.com.qq.office


安装微信

使用文本编辑器打开/etc/pacman.conf
//添加国内包的镜像源
[archlinuxcn]
SigLevel = Never
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

//更新软件列表，并且下载Arch Linux CN PGP keyring
sudo pacman -Syy
sudo pacman -S archlinuxcn-keyring

sudo pacman -S deepin-wine deepin-wine32
sudo pacman -S wine-wechat
```
> 参考资料
- [在Linux或MacOS上运行Windows程序的6个方法](https://www.sohu.com/a/370810786_185201)
- [ArchLinux安装Deepin-Wine-TIM](https://blog.csdn.net/u010255072/article/details/85105986)

### MarkDown预览与编辑器： Typora

---
```
    sudo pacman -S typora
```

### 网易云音乐

+ 方法一：
```
    配置源
        打开/etc/pacman.conf，在末尾加上
    
        [archlinuxcn]
        SigLevel = Optional TrustAll
        Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
    
    安装：
        sudo pacman -S netease-cloud-music

    问题：
    1、无法输入中文
    解决尝试：
    将/opt/netease/netease-cloud-music下的netease-cloud-music.bash改一下就行


```

+ 方法二
```
yay -S netease-cloud-music
```

+ 第三方开发的网易云音乐客户端
```
yay -S iease-music
```

### 洛雪音乐助手: lx-music-desktop

+ [lx-music-desktop github项目地址](https://github.com/lyswhut/lx-music-desktop)

+ Archlinux安装
```
yay -S lx-music-desktop
```

### 讯飞输入法：iflyime

### redis管理工具：another-redis-desktop-manager [aur]

### deepin的计算器：deepin-calculator

### 邮件管理器:thunderbird

```
sudo pacman -S thunderbird
```

### 桌面GUI解压软件：file-roller

### 文件管理器：thunar

```
sudo pacman -S thunar
```

### 文件管理器： nemo
```
sudo pacman -S --noconfirm nemo nemo-python nemo-fileroller nemo-preview nemo-terminal cinnamon-translations
```
> 参考资料
- [安装和使用ArchLiunx超详细教程](https://zhuanlan.zhihu.com/p/141712466)


### 文件管理器：deepin-file-mamager

---
```
    aurman -S deepin-file-manager

    启动;
    dde-file-manager

```

### 终端文件管理器:ranger

---
```
    安装
        sudo pacman -S ranger

    使用：
        ranger

        快捷键命令
        ?           打开帮助手册或列出快捷键打开帮助手册或列出快捷键、命令以及设置项
        l, Enter    打开文件
        j, k        选择当前目录中的文件
        h, l        在目录树中上移和下移

        空格        选中光标所在文件
        cw          重名选中的文件
        yy          复制
        pp          黏贴复制到文件
        :delete     删除
    
    技巧：
        soure range 这样打开ranger退出会自动切换到最后访问的目录位置
```
+ 配置
```
ranger 会创建一个目录 ~/.config/ranger/。可以使用以下命令复制默认配置文件到这个目录:

$ ranger --copy-config=all
了解一些基本的 python 知识可能对定制 ranger 会有帮助。

rc.conf - 选项设置和快捷键
commands.py - 能通过 : 执行的命令
rifle.conf - 指定不同类型的文件的默认打开程序。
rc.conf 只需要包含与默认配置文件不同的部分, 因为它们都会被加载。对于 commands.py，如果你没有包含整个文件，把下面这一行加入到文件起始处：

from ranger.api.commands import *
```

```
使用 ueberzug在ranger 中预览图片
	使用pip3安装ranger:
		pip3 install ranger-fm
	安装ueberzug:
		pip3 install ueberzug
	修改ranger的rc.conf:
	set preview_images_method ueberzug
```
> 参考资料
- [ranger的配置与使用](https://zhuanlan.zhihu.com/p/105731111)
- [ranger无法预览图面](https://zhuanlan.zhihu.com/p/266553673)

### 文件管理器： nautilus

+ 方式二: aur安装
```
yay -S nautilus
```
> 参考资料

## 卓越且功能齐全的 Markdown 编辑器： ：remarkable

```
yay -S remarkable
```
## 对linux用户免费的PDF浏览及编辑器，支持实时预览: masterpdfeditor 

```
yay -S masterpdfeditor
```

## 类似Windows的画图工具: pinta
```
yay -S pinta
```

## 文本比较: meld 

## 基于GTK+的科学计算器： galculator

## 词典软件：  goldendict 

## 鼠标手势:  easystroke  


### 截图软件: scrot

---
```
    安装：
        sudo pacman -S scrot   

    使用：
        scrot [option] [file] -----基本格式 scrot 参数 文件名 
        参数
             -d NUM -------延迟NUM秒后截屏
             -c -----搭配-d,显示延时的倒计时
             -s -------手动框选截图(没法截图,自己体验)
        
             如：scrot -s xxx.png  // 会保存到当前目录下名为xxx.png
```

> 参考资料
- [arch截屏工具--shutter(图形化) / scort(命令)/Peek(GiF)](https://www.jianshu.com/p/008ae8af32de)


### 截图软件：flameshot

```
	sudo pacman -S flameshot
```

> 参考资料
- [分享|在 Linux 下截屏并编辑的最佳工具](https://linux.cn/article-10070-1.html?pr)


### 压缩解压工具：unrar

---
```
	安装
	sudo pacman -S unrar
```

### 压缩解压工具：zip、unzip

---
```
	安装
	sudo pacman -S zip unzip
```

### 压缩解压工具：7zip

---
```
	安装
	sudo pacman -S 7zip

	使用
	压缩
	7z a yajiu.7z yajiu.jip yajiu.png    将yajiu.jpg、yajiu.png压缩为一个7z包
	7z a yajiu.7z yajiu                  将yajiu文件夹压缩成一个7z包

	解压
	7z x yajiu.7z  将yajiu.7z包解压打到压缩包命名的目录下
```

### 终端 项目 npm 管理字符界面:lazynpm

```
yay -S lazynpm
```

### 终端git客户端：lazygit

---
```
    aurman -S lazygit
    或
    yay -S lazygit
    github地址：https://github.com/jesseduffield/lazygit

```

### 终端文件管理器：Vifm

### 文件管理器：rox

---
```
    安装：
        sudo pacman -S rox
    
    使用：
        rox     // 打开窗口
```

### 终端文件管理器：fff

---
```
    sudo pacman -S fff
```

### U盘挂载: udev

---
```
    sudo pacman -S udev
```

### U盘挂载: udevil

---
```
    sudo pacman -S udevil

```

### U盘挂载: udisks

---
```

```
### 下载工具： Flareget

---
```
    aurman -S flareget
```

### 下载工具:uGet


---
```
    sudo pacman -S uget

```

### 迅雷: xunlei 、thunderspeed

1、Linux版本 
```
AUR：
yay -S xunlei-bin

deb版本：
麒麟那边也发布了一个版本，版本号没变，但校验和不一样，据说修复了一些问题。
http://archive.kylinos.cn/kylin/partner/pool/com.xunlei.download_1.0.0.1_amd64.deb

http://archive.kylinos.cn/kylin/partner/pool/com.xunlei.download_1.0.0.1_arm64.deb

```

+ 获取迅雷真实下载地址
```
1、复制迅雷下载地址
如：thunder://QUFodHRwOi8vdmlweHouYm9jYWktenVpZGEuY29tLzIxMDIvU1Pku6QuVEPmuIXmmbDniYgubXA0Wlo

2、去除thunder:// 只取后面的加密字符串
使用base64进行解码,如：
echo -n "QUFodHRwOi8vdmlweHouYm9jYWktenVpZGEuY29tLzIxMDIvU1Pku6QuVEPmuIXmmbDniYgubXA0Wlo" | base64 -di
得到输出AAhttp://vipxz.bocai-zuida.com/2102/SS令.TC清晰版.mp4ZZbase64: 输入无效

3、使用wget、curl、uget或axel等下载工具下载
如： uget http://vipxz.bocai-zuida.com/2102/SS令.TC清晰版.mp4
```

- [下载地址转换原始地址支持迅雷,快车,旋风的下载地址互转. 同时支持fs2you下载地址转换](https://tool.lu/urlconvert/)

### 几乎全能的下载神器：aria2

+ 封装的gui客户端
	- [单纯的Yaaw+内置Aria2](https://github.com/yangshun1029/aria2gui)
	- [ Persepolis Download Manager 官网](https://github.com/persepolisdm/persepolis)

```
yay -S persepolis
```

## 网页视频下载方案:you-get、 FFmpeg

1. 安装 pip
```
curl https://bootstrap.pypa.io/get-pip.py | python3
```

1. 安装 you-get
```
pip install you-get
或
yay -S you-get
```

1. 安装 ffmpeg
```
brew install ffmpeg

如果不下载FFmpeg ，视频和音频是分离的。FFmpeg[11]的转换和字幕功能非常强大。
```
1. 使用

```
you-get 基本操作命令：（可暂时先不看）

usage: you-get [OPTION]... URL...

A tiny downloader that scrapes the web

optional arguments:
  -V, --version  Print version and exit   #显示版本并退出
  -h, --help  Print this help message and exit   #显示基本命令

Dry-run options:  (no actual downloading)   #空运行/预检操作（没有实际下载内容）

  -i, --info  Print extracted information   #显示提取下载信息
  -u, --ur  Print extracted information with URLs   #获得页面所有可下载URL列表
  --json  Print extracted URLs in JSON format   #获得以JSON格式的页面所有可下载信息

Download options:   #下载选项

  -n, --no-merge   Do not merge video part   #不合并视频
  --no-caption   Do not download captions (subtitles, lyrics, danmaku,...)   #不下载其他文件（字幕，歌词，弹幕等）
  -f, --force   Force overwriting existing files   #覆盖存在的文件
  --skip-existing-file-size-check   Skip existing file without checking file size   #
  -F STREAM_ID, --format STREAM_ID   Set video format to STREAM_ID   #选择下载哪种清晰度的视频

  -O FILE, --output-filename FILE   Set output filename   #设定输出文件名
  -o DIR, --output-dir DIR   Set output directory   #设定输出路径
  -p PLAYER, --player PLAYER   Stream extracted URL to a PLAYER   #将提取的真实地址传给播放器
  -c COOKIES_FILE, --cookies COOKIES_FILE   Load cookies.txt or cookies.sqlite   #导入cookies.txt或cookies.sqlite（firefox下使用export-cookies插件）
  -t SECONDS, --timeout SECONDS   Set socket timeout   #设置代理的timeout
  -d, --debug   Show traceback and other debug info   #显示traceback和其他的debug信息
  -I FILE, --input-file FILE   Read non-playlist URLs from FILE   #仅下载链接的视频不下载列表
  -P PASSWORD, --password PASSWORD   Set video visit password to PASSWORD   #
  -l, --playlist   Prefer to download a playlist   #下载列表
  -a, --auto-rename   Auto rename same name different files   #
  -k, --insecure   ignore ssl errors   #

Proxy options:    #代理选项

  -x HOST:PORT, --http-proxy HOST:PORT   Use an HTTP proxy for downloading   #使用HTTP代理下载
  -y HOST:PORT, --extractor-proxy HOST:PORT   Use an HTTP proxy for extracting only   #仅对真实地址视频文件的下载使用HTTP代理
  --no-proxy   Never use a proxy   #不使用代理
  -s HOST:PORT, --socks-proxy HOST:PORT   Use an SOCKS5 proxy for downloading


1 查看视频格式信息

输入：

you-get -i 网址URL   #参数“i”代表“infomation”，查看视频的信息包括：格式、画质、大小等
输出：

2 选择下载的格式

you-get --format=格式
# download-with: you-get --format=****提示的内容，选择不同清晰度进行下载。

例如：我选择清晰度较高的dash-flv720，这块代码如下显示：
you-get --format=dash-flv720 网址URL

3 选择保存的文件夹

you-get -o 文件夹位置 网址URL  

#参数“o”代表“output”，代表放置位置。文件位置用直接拖拽进来即可。


4 下载一个列表的全部视频

you-get --playlist 网址URL


5 将以上4条合成在一条代码中显示：

输入：

you-get --format=dash-flv720 -o /Users/用户名/Vedio\ you-get --playlist 网址URL

#选择的视频格式为dash-flv720，保存的文件夹位置/Users/用户名，文件夹名Vedio you-get
```

+ 例子
```
you-get --format=MPEG-4 -o /Users/whoami/Vedio\  https://v.qq.com/x/cover/mzc00200dfbfsrw.html


you-get --format=MPEG-4 -o /Users/whoami/Vedio\  https://ts.cdn-c55291f64e9b0e3a.com/common/duanshipin/2020-05-25/dsp_61a8784b164c572f7543e1d6a464dd7b_wm/dsp_61a8784b164c572f7543e1d6a464dd7b_wm3.ts

you-get  -i https://ts.cdn-c55291f64e9b0e3a.com/common/duanshipin/2020-05-25/dsp_61a8784b164c572f7543e1d6a464dd7b_wm/dsp_61a8784b164c572f7543e1d6a464dd7b_wm3.ts
```

> 参考资料
- [you-get Github](https://github.com/soimort/you-get)
- [Mac下pip的安装](https://www.jianshu.com/p/dd67e564927a)
- [macOS一键下载You-Get全网站视频](https://zhuanlan.zhihu.com/p/106756882)

- [arch Linux如何下载b站视频](https://jingyan.baidu.com/article/4853e1e5bbd6ae1908f7265e.html)

#### m3u8格式视频下载工具

- [golang 多线程下载直播流m3u8格式的视屏，跨平台](https://github.com/llychao/m3u8-downloader)

+ 安装
```
linux下：
可直接下载编译完的文件，[点击下载](https://github.com/llychao/m3u8-downloader/releases)
将下载下来的m3u8-downloader赋执行权限 sudo chmod +x ./m3u8-downloader
将m3u8-downloader移动到/usr/loacl/bin中，使其全局可用 sudo cp  ./m3u8-downloader /usr/local/bin
```

+ 使用
```
进入网页视频按下f12调出开发者模式，并选择NetWork 标签后输入“.m3u8”进行过滤

简洁使用：./m3u8-downloader  -u=http://example.com/index.m3u8
完整使用：./m3u8-downloader  -u=http://example.com/index.m3u8 -o=example -n=16 -ht=apiv1

参数说明：
默认情况只需要传u参数,其他参数保持默认即可。 部分链接可能限制请求频率，可根据实际情况调整 n 参数的值。

Usage of m3u8-linux-amd64:
  -c string
        自定义请求cookie
  -ht http(s):// + url.Host + path.Dir(url.Path) 设置getHost的方式（共两种 apiv1 和 apiv2）, 默认 apiv1
        设置getHost的方式(apiv1: http(s):// + url.Host + path.Dir(url.Path); apiv2: `http(s)://+ u.Host` (default "apiv1")
  -n int
        下载线程数(max goroutines num) (default 16)
  -o string 
        自定义文件名(默认为output) (default "output")
  -u string
        m3u8下载地址(http(s)://url/xx/xx/index.m3u8)

```

> 参考资料
- [m3u8流网站如何获得m3u8地址？](https://www.zhihu.com/question/302692755/answer/1197296821)
- [如何提取视频的m3u8地址？](https://jingyan.baidu.com/article/f3ad7d0f1bc87109c3345b82.html)


### 下载工具：pyIDM

+ 安装
```
pip install pyIDM
```

> 参考资料
- [Linux下安装PyIDM – IDM的开源替代品（Internet Download Manager）](https://m.linuxidc.com/Linux/2020-02/162467.htm)

### 米聊

```
yay -S mitalk
```



### 文件对比同步：freefilesync

---
```
    安装方式：aurman安装freefilesync
        aurman -Ss  freefilesync      // 搜索查看
        aurman -S   freefilesync      // 安装freefilesync
    
    dwm打开 FreeFileSync

    安装方式：二进制文件安装

    安装方式：源码编译安装

```

### Firefox浏览器

---
```
    
    pacman -S firefox                  // 安装FireFox浏览器
    
    pacman -S firefox-i18n-zh-cn       // 安装中文语言包 

```

> 参考资料

+ [Arch Linux安装Firefox 火狐中文版](https://blog.csdn.net/weixin_30895603/article/details/99177819)

### Chrome浏览器

---
```

    sudo pacman -S chromium

```

> 参考资料

+ [在archlinux中安装chrome](https://www.cnblogs.com/goodloop/archive/2010/11/11/1875238.html)

### pdf阅读器

---
```
    sudo pacman -S evince

    使用
       evince
```


### WPS office 国内版

---
```
    yay -S wps-office-cn          // 安装wps国内版
    yay -S wps-office-mui-zh-cn   // 安装中文语言包

    yay -S ttf-wps-fonts          // 安装一些字体 symbol.ttf webdings.ttf wingding.ttf wingding2.ttf wingding3.ttf  MTExtra.ttf

```

### 音乐播放服务器:mpd

MPD（音乐播放器守护程序）是具有服务器-客户端体系结构的音频播放器。它可以使用很少的资源来播放音频文件，组织播放列表并维护音乐数据库。为了与之交互，需要一个单独的客户端

```
sudo pacman -S mpd 
# > 安装完需要去配置（官方配置指南与例子文件在 /usr/share/doc/mpd/mpdconf.example）
启动（调试模式）
mpd --stderr --no-daemon --verbose 
或mpd 配置文件 --stderr --no-daemon --verbose 
注： 默认是启动 ~/.mpd/mpd.conf .若sudo 或root用户启动默认是/etc/mpd.conf


将mpd用户添加到wheel、root、audio的组，并授予组对用户目录的执行权限。通过这种方式，mpd用户有权打开用户目录：
＃gpasswd -a mpd wheel 
＃gpasswd -a mpd root 
＃gpasswd -a mpd audio 
```

```
# mpd客户端mpc
sudo pacman -S mpc

# mpd客户端ncmpc
sudo pacman -S ncmpc

```

> 参考资料
- [mdp官方指南手册](https://www.musicpd.org/doc/html/user.html)
- [ArchLinux的mpd手册](https://wiki.archlinux.org/index.php/Music_Player_Daemon)

### mpd音乐客户端：mpc


+ 安装mpd客户端mpc
```
sudo pacman -S mpc
```
+ 使用
```
$ mpc toggle #启动和暂停
$ mpc play # 播放
$ mpc pause # 暂停
$ mpc stop # 关闭
$ mpc next #下一首
$ mpc prev #前一首

$ mpc repeat on #启用重复播放
$ mpc consume on #播放完该首音乐会从播放列表中移除该歌曲
$ mpc single on  #播放完该首音乐会停止，若配合repeat才会重复
$ mpc randon on  #下一首歌曲随机

$mpc volume +20 #提高20的音量

$mpc seek +1 #快进1%
$mpc seek -1 #快退1%

一些参数可以组合使用，比如：
$ mpc repeat on
$ mpc single on
这会对单曲进行重复播放

$ mpc repeat on
$ mpc randon on
这会对进行列表重复播放
```


### mpd音乐客户端：ncmpc

### 视频播放器：mpv

```
    sudo pacman -S mpv

    使用：
        mpv 视频名
```

```
mpv 默认配置在性能较弱的电脑上应当也能正常工作。 然而，如果你有性能更好的显卡，通过配置 mpv 可以得到更好的使用体验 (经过配置，mpv潜力无穷！)。 如果需要配置，只需要新建几个配置文件即可 (这些配置文件默认不存在).

Note: 配置文件/etc/mpv从系统范围内读取，配置文件~/.config/mpv从用户范围内读取。 (除非你设定了 environment variable XDG_CONFIG_HOME环境变量), 用户配置文件的优先级要高于系统配置文件。而直接在命令行使用参数的优先级又高于用户配置文件。建议使用用户配置文件，因为在配置过程中可能需要不断尝试和试错。

mpv 提供了范例配置文件。 把它们当做你的第一步吧:

# 复制默认配置文件到用户配置下进行自定义修改
$ cp -r /usr/share/doc/mpv/ ~/.config/
mpv.conf 是 mpv'的主要配置文件, 
input.conf 是按键绑定的配置文件。
通读它们两个，以了解它们的工作原理以及可用的选项。
```

> 参考资料
+ [有什么好用的播放器吗](http://tieba.baidu.com/p/5461375837)
+ [mpv (简体中文)-wiki](https://wiki.archlinux.org/index.php/Mpv_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

### 音频播放器: moc

```
    sudo pacman -S moc

	使用
		进入音乐文件所在文件夹
		在命令行终端输入命令 mocp

		或
		mocp 音乐文件所在目录路径
```

### 终端音乐播放器: cmus

---
```
    sudo pacman -S cmus

    使用：
        常用快捷键设置类似于vim，若有相关经验会感到很舒适。
        
        添加歌曲
            在任一界面，输入":add 文件名或目录名"，则会将对应的歌曲文件或目录中的所有歌曲文件全部添加到库中。
        
        播放歌曲
            选择任一歌曲，按下Enter或c即可播放或暂停。
        
        播放暂停歌曲，c或Enter
        随机播放，s
        降低音量，-
        提高音量，=
        添加到播放队列，e
        显示当前音乐文件的位置，I
        导入音乐	:a /music/pach/
        播放或重播音乐	x
        播放下一首音乐	b
        播放上一首音乐	z
        激活随机播放	s 
        清空当前文件列表	:clear
        保存播放列表	:save /path/to/playlist
        加载播放列表	:load /path/to/playlist
        在7个不同的功能界面之间切换	数字键1-7
		Tab键在有两个的子窗口间切换
        退出播放器	q
```

+ 问题记录： root用户打开能播放出声音，普通用户播放时报错： Error：Opening audio device:No such device
	```
	解决：
	1. 查看声音设备用户和用户组 ls -al /dev/snd
	2. 看当前用户是否在设备的用户组中，没有则添加
	修改用户下的cmus配置文件 vim ~/.config/cmus/rc（无此文件则增加）

	set output_plugin=alsa
	set dsp.alsa.device=default
	set mixer.alsa.device=default
	set mixer.alsa.channel=Master

	```


> 参考资料
+ [Cmus终端音乐播放器](https://www.jianshu.com/p/1d22f569918a)
+ [命令行音乐播放器 CMus](https://www.cnblogs.com/tsdxdx/p/7221109.html)
+ [cmus - ArchWiki](https://wiki.archlinux.org/index.php/Cmus）
+ [ArchLinux安装完没有声音之解决办法_操作系统_weixin_33709609的博客-CSDN博客](https://blog.csdn.net/weixin_33709609/article/details/89606443)

### 文本编辑器:Sublime text 3

+ aur安装方法:
```
	yay -S sublime-text
```

+ 2020年Sublime text 3注册码
```
----- BEGIN LICENSE -----
Member J2TeaM
Single User License
EA7E-1011316
D7DA350E 1B8B0760 972F8B60 F3E64036
B9B4E234 F356F38F 0AD1E3B7 0E9C5FAD
FA0A2ABE 25F65BD8 D51458E5 3923CE80
87428428 79079A01 AA69F319 A1AF29A4
A684C2DC 0B1583D4 19CBD290 217618CD
5653E0A0 BACE3948 BB2EE45E 422D2C87
DD9AF44B 99C49590 D2DBDEE1 75860FD2
8C8BB2AD B2ECE5A4 EFC08AF2 25A9B864
------ END LICENSE ------​
```

+ sublime-merge安装

```
方法一：
Install the GPG key:

	curl -O https://download.sublimetext.com/sublimehq-pub.gpg && sudo pacman-key --add sublimehq-pub.gpg && sudo pacman-key --lsign-key 8A8F901A && rm sublimehq-pub.gpg

Select the channel to use:

Stable
	echo -e "\n[sublime-text]\nServer = https://download.sublimetext.com/arch/stable/x86_64" | sudo tee -a /etc/pacman.conf
Dev
	echo -e "\n[sublime-text]\nServer = https://download.sublimetext.com/arch/dev/x86_64" | sudo tee -a /etc/pacman.conf

Update pacman and install Sublime Merge

	sudo pacman -Syu sublime-merge


方法二：

	yay -S sublime-merge
```

+ 问题记录1:archlinux 导入密钥时出现问题
```
:: 正在用 gpg 导入密钥...
gpg: 警告：服务器 ‘dirmngr’ 比我们的版本更老 （2.2.23 < 2.2.24）
gpg: 注意： 过时的服务器可能缺少重要的安全修复。
gpg: 注意： 使用 “gpgconf --kill all” 来重启他们。
gpg: 从公钥服务器接收失败：一般错误
导入密钥时出现问题

解决办法：
sudo -i dirmngr < /dev/null
准确来说,这个命令的作用是让dirmngr正确的启动.
```

### 影音播放器：MPlayer


---
```

    sudo pacman -S mplayer   // 安装完之后是字符界面的

    sudo pacman -S gnome-mplayer  // 图形前端
```

### 托盘程序：trayer

---
```
    sudo pacman -S trayer

    使用
        trayer

```

## 美化操作

### grub美化

```

    1、下载主题
        可以到 gnome-look(https://www.gnome-look.org/) 选择喜欢的主题下载。
    
        pacman -S wget  // 安装下载工具
        pacman -S unzip zip // 安装zip压缩与解压工具
        例如： wegt https://codeload.github.com/gustawho/grub2-theme-breeze/zip/master

    2、解压缩主题包
        sudo tar -xf 主题包名 
        复制到主题目录
        sudo cp -r 主题包名 /boot/grub/themes/  

        或者执行主题包下的安装脚本命令
    3、修改配置文件
        sudo vim /etc/grub.d/00_header
        添加如下内容：

        GRUB_THEME="/boot/grub/themes/主题包/theme.txt"
        GRUB_GFXMODE="1024x768x32"

    4、更新grub配置文件

        sudo grub-mkconfig -o /boot/grub/grub.cfg


```

```
修改grub启动顺序

1. 打开配置文件
sudo vim /etc/default/grub

2. 修改第一行附近的GRUB_DEFAULT的值

陪置参数说明如下：
# If you change this file, run ’update-grub‘ afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n ’Simple configuration‘

GRUB_DEFAULT=0
#属性名：默认启动项（就是我要的开机默认启动系统）
#值说明：
#   数字：从0开始（按照开机选择界面的顺序对应）
#   saved:默认上次的启动项

#GRUB_HIDDEN_TIMEOUT=0
#属性名：是否隐藏菜单（grub2不再使用）
#值说明：0：不隐藏，1：隐藏

GRUB_HIDDEN_TIMEOUT_QUIET=true
#属性名：是否显示等待倒计时
#值说明：true：不显示，false：显示

GRUB_TIMEOUT=10
#属性名：进入默认启动项的等候时间
#值说明：单位：秒，默认10秒，-1表示一直等待

GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`

GRUB_CMDLINE_LINUX_DEFAULT=”quiet splash“
#属性名：内核启动参数的默认值
#值说明：quiet splash为不显示启动信息，安静的启动，如值为空则显示启动信息

GRUB_CMDLINE_LINUX=”“
#属性名：手动添加内核启动参数
#值说明：默认为空，可以添加你需要的参数，以 “name=value” 的格式添加，多个参数用空格隔开
#例如：GRUB_CMDLINE_LINUX=”name1=value1 name2=value2“

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM=”0x01234567,0xfefefefe,0x89abcdef,0xefefefef“

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console
#属性名：是否使用图形介面
#值说明：默认使用图像界面，去掉前面的“#”则使用控制台终端

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo’

#GRUB_GFXMODE=640x480
#属性名：图形界面分辨率
#值说明：分辨率啦（还要怎么说明），修改时记得去掉前面的“#”

# Uncomment if you don‘t want GRUB to pass ”root=UUID=xxx“ parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true
#属性名：grub命令是否使用UUID
#值说明：不知道是干什么的，不常用（如果你知道，欢迎留言，谢谢）

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY=”true“
#属性名：是否创建修复模式菜单项
#值说明：true:禁用，false：使用，默认false

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE=”480 440 1“
#属性名：启动时发出哔哔声
#值说明：默认不发声，去掉“#”则发声，值是什么意思不明白（应该是发出声音方式吧）



3. 修改完 更新 
    grub-mkconfig -o /boot/grub/grub.cfg
    
```

---
```
    问题：grub分辨率问题

    需要到grub启动菜单哪里按c，进入grub命令行。
    efi启动的输入videoinfo，其他的输入vbeinfo，反正这两个命令试一下，哪个有输出用哪个就行，结果是一样的。
    输入之后会列出支持的分辨率。
    找到合适的分辨率填到GRUB_GFXMODE配置里面就行了，然后重新生成grub配置。

    到 /etc/default/grub
    GRUB_GFXMODE=1280x1024,1024x768,auto

    和 /etc/grub.d/00_header
    GRUB_TERMINAL_OUTPUT="gfxterm"
    GRUB_GFXMODE=1280x1024,1024x768,auto
```

> 参考资料

+ [Grub2主题修改和美化--------Linux&Windows](https://blog.csdn.net/GenuineMonster/article/details/83685479)
+ [Archlinux 修改 grub 主题](https://www.jianshu.com/p/7091972678b4)

+ [Deepin 15.8系统Grub菜单分辨率低的原因及解决方案](https://ywnz.com/linuxjc/3587.html)

+ [怎么改grub菜单的分辨率？我改了不生效呀。](https://tieba.baidu.com/p/5880408866?red_tag=3413271895)

+ [调整 GRUB 启动菜单和虚拟终端的字体大小](https://cnzhx.net/blog/adjust-font-size-of-grub-and-console/)

+ [ArchLinux 安装/配置/美化 --- VMware 篇](https://www.mivm.cn/archlinux-vmware/)

### 使用screenfetch来展示Linux发行版logo

---
```
    安装：
        sudo pacman -S screenfetch
    
    使用：
        screenfetch    // 可以看见用ASCII字符组合而成的logo以及系统信息。

        设置打开终端自动执行screenfetch命令，即在每次打开终端的时候能自动看见logo以及系统信息
        sudo vim ~/.bashrc   // 在文件最后一行加入 screenfetch


```

> 参考资料

+ [如何在终端里展示Linux发行版的logo及系统信息](https://www.linuxdashen.com/%E5%A6%82%E4%BD%95%E5%9C%A8%E7%BB%88%E7%AB%AF%E9%87%8C%E5%B1%95%E7%A4%BAlinux%E5%8F%91%E8%A1%8C%E7%89%88%E7%9A%84logo%E5%8F%8A%E7%B3%BB%E7%BB%9F%E4%BF%A1%E6%81%AF)

### 装逼神器 Linux发行版logo： neofetch

### 动物说 cowsay

```
	sudo pacman -S cowsay

	cowsay xxxx
	或
	echo "xxxxx" | cowsay

	查看全部图案
	$ for i in $(cowsay -l); do cowsay -f $i "$i"; done

	随机动物
	echo "xxxxxx" | cowsay -f $( ls /usr/share/cows | shuf -n 1)
```

> 参考资料

- [CentOS安装fortune+cowsay实现cool登录欢迎语 - 简书](https://www.jianshu.com/p/d0585ce8e78c)

### 动物说: xcowsay

### 名言： forfune

```
	sudo pacman -S fortune-mod


```

### 输出彩虹特效的命令行工具:lolcat

```
	sudo pacman -S lolcat

	ls -al  | lolcat
```

> 参考资料
- [centos7 fortune+cowsay+lolcat美化初始终端 - 夏天公子 - 博客园](https://www.cnblogs.com/chenglee/p/10475576.html)

### 实时显示键盘输入:screenkey

```
	aurman -S screenkey
``` 

```
问题记录: 切换成中文时会被强制转回 无法正常输入中文

```
> 参考资料

- [linux 下在屏幕上显示按键的小工具_运维_Dr.Android-CSDN博客](https://blog.csdn.net/iteye_8750/article/details/82210416)

### 窗口渲染器/合成器

---
```
    带毛玻璃特效的Compton
    源码地址：https://github.com/tryone144/compton
```

### 窗口渲染器/合成器：xcompmgr

---
```
	# 进入桌面时启动
	# xcompmgr -c -C -t-5 -l-5 -r4.2 -o.55 &
	xcompmgr -Ss -n -Cc -fF -I-10 -O-10 -D1 -t-3 -l-4 -r4 &
	
	# 辅助单独设置窗口透明度
	transset-df  0.6
```

### 键盘按键出声软件 tickeys

---
```
    按qaz123可把设置调出

``` 

> 参考资料

+ [tickeys官网](http://www.yingdev.com/projects/tickeys)

### 登录管理器:SLiM

---
```
    安装：
        sudo pacman -S slim

    配置：
        sudo systemctl enable slim  // 设置开机启动登录管理器

        sudo systemctl start slim // 现在直接启动

        sudo systemctl disable slim  // 设置开机不启动启动登录管理器

        安装slim-themes软件包：
            # pacman -S slim-themes archlinux-themes-slim
        软件包 archlinux-themes-slim 包含了多个不同主题。可以在 /usr/share/slim/themes 中查看。在to see the themes available. Enter the theme name on the current_theme line in /etc/slim.conf:
        编辑/etc/slim.conf中的current_theme那行，将"default"改为你想要的主题名：
            #current_theme       default
            current_theme       archlinux-simplyblack

    使用随机主题
        在slim.conf中的current_theme行，添加多个主题，使用,分隔就可以使用随机主题了。

```

### 登录管理器：lightdm

---
```
LightDM 相关操作
切换命令行： alt-ctrl-F2  // 进入tty2界面
LightDM 日志： /var/log/lightdm。
关停 LightDM： sudo systemctl stop lightdm
启动 LightDM： sudo systemctl start lightdm

设置开机启动 sudo systemctl enable lightdm
```

### lightdm 进入后引导dwm、i3等桌面

```
	sudo pacman -S lightdm lightdm-deepin-greeter
```


> 参考资料

+ [LightDM 轻量级桌面显示管理器](https://blog.csdn.net/jacolin/article/details/50729178?utm_source=distribute.pc_relevant.none-task)


### 登录管理器: sddm

```
	sudo pacman -S sddm

	将 dwm.decktop、i3.decktop、deepin.desktop 复制到 /usr/share/xsessions/下，才可在登录管理器页面出现dwm、i3、deepin等的登录选项
	
```

+ dwm.desktop
```
[Desktop Entry]
Name=dwm
Comment=improved dynamic tiling window manager
Exec=dwm
TryExec=dwm
Type=Application
X-LightDM-DesktopName=dwm
DesktopNames=dwm
Keywords=tiling;wm;windowmanager;window;manager;

```

+ i3.desktop
```
[Desktop Entry]
Name=i3
Comment=improved dynamic tiling window manager
Exec=i3
TryExec=i3
Type=Application
X-LightDM-DesktopName=i3
DesktopNames=i3
Keywords=tiling;wm;windowmanager;window;manager;

```

+ deepin.desktop
```
[Desktop Entry]
Name=Deepin
Comment=Deepin Desktop Environment
Exec=/usr/bin/startdde
TryExec=/usr/bin/startdde

```

---
```
	SDDM 的默认配置文件为 /usr/lib/sddm/sddm.conf.d/default.conf。
	要修改配置，请在 /etc/sddm.conf.d/ 目录下创建配置文件。	
	
	创建配置文件/etc/sddm.conf
		~~sudo sddm --example-config > /etc/sddm.conf~~
		sudo sddm --example-config > ~/sddm.conf
		sudo cp ~/sddm.conf /etc/sddm.conf

	主题设置
		在 [Theme] 小节更改主题设置。
		当前主题
		通过 Current 的值设置当前主题，例如 Current=archlinux-simplyblack。

	编辑主题
		默认 SDDM 主题目录为 /usr/share/sddm/themes/。你可以添加你的自制主题到该目录下一个单独的子目录中。注意 SDDM 要求这些子目录的名字要与主题的名字一致。可以通过研究已安装的文件来更改或创建属于你的主题。

	测试（预览）主题
		如果需要，你可以预览一个 SDDM 主题。如果你想知道一个主题看起来怎么样，或是想要编辑一个主题后在不必登出的情况下观察改动的效果，那么这将会非常有用。你可以运行下面的命令：

	
	$ sddm-greeter --test-mode --theme /usr/share/sddm/themes/breeze


	用户图标（头像）
		SDDM 对于每个用户从相应的 ~/.face.icon 目录下读取 PNG 图片格式的用户图标（即“头像”），或者从 SDDM 配置文件中由 FacesDir 为所有用户定义的共有位置。配置设置可以通过修改 /etc/sddm.conf 文件被直接替换，更好的方法是创建一个位于 /etc/sddm.conf.d/ 下的文件来修改，例如 /etc/sddm.conf.d/avatar.conf。

		要使用 FacesDir 选项来确定头像位置，即在配置文件中 FacesDir 所确定的位置为每一个用户放置一个 PNG 图片，命名如 username.face.icon。FacesDir 默认的路径为 /usr/share/sddm/faces/。你可以更改默认 FacesDir 目录以合乎你的要求。下面是一个例子：

		/etc/sddm.conf.d/avatar.conf
		[Theme]
		FacesDir=/var/lib/AccountsService/icons/

```

> 参考资料

- [SDDM (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/SDDM_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%BD%93%E5%89%8D%E4%B8%BB%E9%A2%98)


## 设备管理

---
```
    用rfkill命令管理蓝牙和wifi
        rfkill list

    #关闭编号0的设备
        rfkill block 0

    #打开编号0的设备
        rfkill unblock 0
```


### 设备：蓝牙配置与链接

```
	安装蓝牙工具包
		sudo pacman -S bluez bluez-utils

		蓝牙音频设备包 sudo pacman -S pulseaudio-bluetooth pulsemixer
		蓝牙控制：命令行控制安装bluez-utils，使用参考通过命令行工具配置蓝牙；或者安装蓝牙图形界面工具如blueberry。

	使用
		启动蓝牙服务
			启动         sudo systemctl start bluetooth
			关闭         sudo systemctl stop bluetooth 
			开机启动     sudo systemctl enable bluetooth 
			关闭开机启动 sudo systemctl disable bluetooth 

		进入蓝牙控制台	sudo bluetoothctl
			            agent default
						agent on
			开启        power on
			扫描        scan on
			            pair MAC_address     如: pair 00:10:20:30:40:50
						或 trus MAC_address  使用没有PIN设备用此
						connect MAC_address  如: connect 00:10:20:30:40:50 

```


+ 图形界面蓝牙管理器全功能蓝牙管理器 Blueman

```
	安装
		sudo pacman -S blueman
		sudo pacman -S pulseaudio pavucontrol 
		sudo pacman -S pulseaudio-bluetooth pulsemixer	
	
	使用
		打开Blueman 搜索蓝牙并连接 ：
			运行   blueman-apple
			再运行 blumeman-manager
			在打开的图形界面中选择蓝牙设备连接

		运行 pulsemixer 
			切换设备输出为蓝牙耳机（可实现不同设备播放电影音或这播放音乐）
		
```

> 参考资料
+ [Bluetooth (简体中文)](https://wiki.archlinux.org/index.php/Bluetooth_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
+ [Blueman](https://wiki.archlinux.org/index.php/Blueman)
+ [archlinux 蓝牙耳机没有声音](https://www.cnblogs.com/lemos/p/11515662.html)
+ [Linux安装驱动并使用Blueman连接蓝牙耳机的详细介绍（图文）](https://www.php.cn/linux-364011.html)

## 剪贴板：xclip

---
```
	Linux系统里存在两个剪切板：
		一个叫做选择缓冲区(X11 selection buffer)，
		另一个才是剪切板(clipboard)。
		选择缓冲区是实时的，当使用鼠标或键盘选择内容时，内容已经存在于选择缓冲区了，这或许就是选择缓冲区的由来吧。

	安装
		sudo pacman -S xclip
	
	使用
		查看选择缓冲区
		xclip -out
		
		使用下面的命令查看剪切板的内容：
		$ xclip -out -sel clipboard

		可以使用鼠标中键或键入Shift+Insert来粘贴选择缓冲区的内容。
		但对于有些GUI程序，比如gedit，只能通过鼠标中键调用选择缓冲区的内容，使用Shift+Insert的话，调用的是剪切板的内容。

		剪切板和Windows的剪切板类似，在选择文字内容后，执行Ctrl + c或在菜单里选择‘复制’的话，这时内容才存放到剪切板里。
		而使用剪切板的内容，则是Ctrl+v。trl+v。 但在有些情况下，比如gnome-terminal，不能直接使用Ctrl+c，Ctrl+v，这时就要用Shift+Ctrl+c，Shift+Ctrl+v代替。
```

> 参考资料
- [linux中的剪贴板用法，实现vim中原格式粘贴_运维_大海蓝天的专栏-CSDN博客](https://blog.csdn.net/dahailantian1/article/details/78584887)
- [面向Linux的10款最佳剪贴板管理器_Linux新闻_Linux公社-Linux系统门户网站][https://www.linuxidc.com/Linux/2016-05/131263.htm)


## 翻墙工具： lantern


## 有趣的命令

### screenfetch 在linux系统下显示系统信息


### sl 跑火车

### cmatrix 下数字雨

### figlet 放大字符串

### toilet 放大字符串的升级版本

### asciiquarium 水族馆

### espeak 朗读句子

### oneko 跟着鼠标的小老鼠

### fortune 人生语句

### baster 俄罗斯方块

### 后台程序操作

---
```

    一、&
    加在一个命令的最后，可以把这个命令放到后台执行，如
        watch  -n 10 sh  test.sh  &  #每10s在后台执行一次test.sh脚本

    二、ctrl + z
    可以将一个正在前台执行的命令放到后台，并且处于暂停状态。
    
    三、jobs
    查看当前有多少在后台运行的命令
    
    jobs -l选项可显示所有任务的PID，jobs的状态可以是running, stopped, Terminated。但是如果任务被终止了（kill），shell 从当前的shell环境已知的列表中删除任务的进程标识。
    
    四、fg
    将后台中的命令调至前台继续运行。如果后台中有多个命令，可以用fg %jobnumber（是命令编号，不是进程号）将选中的命令调出。
    
    五、bg
    将一个在后台暂停的命令，变成在后台继续执行。如果后台中有多个命令，可以用bg %jobnumber将选中的命令调出。
    
    六、kill
    法子1：通过jobs命令查看job号（假设为num），然后执行kill %num
    法子2：通过ps命令查看job的进程号（PID，假设为pid），然后执行kill pid
    前台进程的终止：Ctrl+c
    
    七、nohup
    如果让程序始终在后台执行，即使关闭当前的终端也执行（之前的&做不到），这时候需要nohup。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。关闭中断后，在另一个终端jobs已经无法看到后台跑得程序了，此时利用ps（进程查看命令）
    
    ps -aux | grep "test.sh"  #a:显示所有程序 u:以用户为主的格式来显示 x:显示所有程序，不以终端机来区分
    
```
> 参考资料

+ [linux后台运行和关闭、查看后台任务](https://www.cnblogs.com/kaituorensheng/p/3980334.html#_label2)

### 日历

---
```
    新历
        cal 

    农历日历安装：
        aurman -S ccal
    使用：
        ccal -u          " 显示 当月日历
        ccal -u 2020     " 显示 2020年 的所有日历
        ccal -u 3 2020   " 显示 2020年3月的日历
```

---
```
    日历特效（需要依赖实用程序（cal/ccal、box/boxes、lolcat、gawk 等），还需要使用支持 Unicode 的终端仿真器。
    第一版：日历加外框
    cal | boxes -d diamonds -p a1l4t2 

    ccal -u | boxes -d diamonds -p a1l4t2 

    第二版：日历加外框+卷轴外框
    cal | boxes -d diamonds -p a1t2l3 | boxes -a c -d scroll        

    ccal -u | boxes -d diamonds -p a1t2l3 | boxes -a c -d scroll        

    第三版：日历加外框+卷轴外框+颜色+下雪特效
    clear;cal|boxes -d diamonds -p a1t2l3|boxes -a c -d scroll|lolcat;sleep 3;while :;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 0.1;done|gawk '{a[$3]=0;for(x in a) {o=a[x];a[x]=a[x]+1;printf "\033[%s;%sH ",o,x;printf "\033[%s;%sH%s \033[0;0H",a[x],x,$4;}}'

    clear;ccal -u|boxes -d diamonds -p a1t2l3|boxes -a c -d scroll|lolcat;sleep 3;while :;do echo $LINES $COLUMNS $(($RANDOM%$COLUMNS)) $(printf "\u2744\n");sleep 0.1;done|gawk '{a[$3]=0;for(x in a) {o=a[x];a[x]=a[x]+1;printf "\033[%s;%sH ",o,x;printf "\033[%s;%sH%s \033[0;0H",a[x],x,$4;}}'

```

> 参考资料
+ [在 Linux 命令行中规划你的假期日历 | Linux 中国](https://blog.csdn.net/F8qG7f9YD02Pe/article/details/86746815)

### 模糊搜索工具：fzf

---
```
    Fzf是一款小巧，超快，通用，跨平台的命令行模糊查找器 
    安装：
        方式一：
            sudo pacman -S fzf
        方式二：
            git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
            cd ~/.fzf/
            ./install

    配置：
    在~/.zshrc 或 ~/.bashrc文件中加入
    #预览与窗口大小配置
    export FZF_DEFAULT_OPTS="--height 49% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -500'"

    使用：
    0、原生使用
        fzf默认会从STDIN读入数据，然后将结果输出到STDOUT
            find * -type f | fzf > selected
        上面命令从find的搜索结果中读入，输出到文件selected中

        在finder（输出交换窗口）里，
            Ctrl-J/Ctrl-K/Ctrl-N/Ctrlk-N可以用来将光标上下移动
            Enter键用来选中条目， Ctrl-C/Ctrl-G/Esc用来退出
            在多选模式下（-m), TAB和Shift-TAB用来多选
                Mouse: 上下滚动， 选中， 双击； Shift-click或shift-scoll用于多选模式

        fzf默认全屏模式，你可以加参数定制高度
            fzf --height 40%
            也可以通过$FZF_DEFAULT_OPTS来设定默认值(在配置文件如:~/.bashrc、～/.zshrc中加入)
            export FZF_DEFAULT_OPTS='--height 40% --reverse --border'

        fzf默认会以“extened-search"模式启动
           可以用fzf -e做精确匹配
           |可以做or匹配， 比如:
            ^core go$|rb$|py$    表示以core开头，以go或rb或py结尾的 
    
    1、在Bash、Zsh中使用模糊(创建命令别名)

        # fzf 配置
        #export FZF_DEFAULT_OPTS="--height 49% --layout=reverse --preview '(highlight -O ansi {} || cat {}) 2> /dev/null | head -500'"
        # 只加加文本预览其他用默认
        #export FZF_DEFAULT_OPTS="--preview ' cat {}' "
        
        # 使用fd来得到文件列表 并加预览
            
        ## 自定义命令别名
        # alias 法
        #alias xiexiela="ls -al --color=auto"
        #alias vimf="vim $(fzf)"
        # zsh函数法
        vimf(){
            # vim $(fzf --preview 'cat {}')
            vim $(fzf)
        }
        nvimf(){
            # nvim $(fzf --preview 'cat {}')
            nvim $(fzf)
        }

		# 使用cd fd fzf 查找与切换目录
		cdf(){
		   # cd $(fd -p '/' -t d | fzf )
		   # cd $(fd -p ’/‘ -t d | fzf --preview 'ls -alh {}')
		   cd $(fd -p ＇/＇ -t  d | fzf --height=60% --preview 'ls -ahl {}' )
		}

         
    2、在vim中使用fzf（需要fzf插件）
       安装fzf.vim
       a、与ripgrep搭配使用        
           在.vimrc配置中中加入

            let g:rg_command = '
              \ rg --column --line-number --no-heading --fixed-strings --ignore-case --no-ignore --hidden --follow --color "always"
              \ -g "*.{js,json,php,md,styl,jade,html,config,py,cpp,c,go,hs,rb,conf}"
              \ -g "!{.git,node_modules,vendor}/*" '
            command! -bang -nargs=* Frg call fzf#vim#grep(g:rg_command .shellescape(<q-args>), 1, <bang>0)
           这样在vim命令模式下输入：Frg 关键词或省缺为空 就可以进行检索相关

    3、搭配fd使用(需要安装fd  安装命令：sudo pacman -S fd)
       递归查找全盘文件夹并进入
       cd $(fd --full-path '/' -d | fzf )
       
```

---
```
    查看alias
    alias
    如果想要删除在终端中临时设置的别名，可以使用 unalias 命令。
    unalias gerp

    如果想要持久保存命令别名，可以将命令别名放在用户主目录的 .bashrc 文件中。
    注：这里看使用的终端
```

> 参考资料
+ [Ubuntu中安装fzf ](https://www.dazhuanlan.com/2019/12/13/5df34f13961b2/)
+ [模糊搜索神器fzf](https://segmentfault.com/a/1190000011328080)
+ [fzf 命令行文本增强工具使用](https://www.jianshu.com/p/8acfc103fa2f)
+ [模糊搜索神器FZF番外篇](https://segmentfault.com/a/1190000016186043)
+ [Linux添加alias简化命令](https://www.cnblogs.com/YMaster/p/9788938.html)
+ [Windows 高效写前端技巧(环境配置篇)](https://www.jianshu.com/p/c65a315f77d0)
+ [CLI命令行模糊搜索的超强大工具：fzf](https://www.jianshu.com/p/d64553a37d69)
+ [Fuzzy finder(fzf+vim) 使用全指南](https://blog.csdn.net/weixin_34256074/article/details/88758143)

### 安装jdk环境  

---  
```  

    1. 从官网下载jdk1.8的tar包  
        注：本次使用包为 jdk-8u211-linux-x64.tar.gz  

    2. 将下载的jdk的tar包解压到/usr/local/  
        tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/  

        //x : 从 tar 包中把文件提取出来  
        //z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压  
        //v : 显示详细信息  
        //f xxx.tar.gz :  指定被处理的文件是 xxx.tar.gz,这几个连用的时候f必须放在最后  
        //-C：指定需要解压到的目录。  

    3. 给解压后的文件创建一个软链接便于以后的版本更替  
        cd /usr/local/  
        ln -s jdk1.8.0_211/ java  

    4. 在/etc/profile中最下面添加java环境配置:  
        vim /etc/profile  

        export JAVA_HOME=/usr/local/java  
        export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib  
        export PATH=$PATH:$JAVA_HOME/bin  

    5、启动/etc/profile文件中的配置:  
        source /etc/profile  

        注意:普通用户登陆系统然后在shell中切换到root启动配文件后再切换回普通用户后配置文件对于普通用户并没有起效果.  
        此时可以注销或者在普通用户环境下再启动一次配置文件.  

    5.简单的java程序测试，查看版本号  
        java -version  

```  

> 参考资料  
+ [Linux使用tar包安装jdk1.8](https://blog.csdn.net/liuchonghua/article/details/82698265)  


### IDE：Idea

---
```
    1. 从官网下载idea的tar包  
        注：本次使用包为

    2. 将下载的idea的tar包解压到/usr/local/  
        tar -zxvf idea-IU-xx.tar.gz  -C /usr/local/  

        //x : 从 tar 包中把文件提取出来  
        //z : 表示 tar 包是被 gzip 压缩过的，所以解压时需要用 gunzip 解压  
        //v : 显示详细信息  
        //f xxx.tar.gz :  指定被处理的文件是 xxx.tar.gz,这几个连用的时候f必须放在最后  
        //-C：指定需要解压到的目录。  

	3. 启动idea，进入解压的idea的文件夹的下的./bin/idea.sh

	4. 加入应用列表，以便dmenu快速打开
		查看执行 echo $PATH 查看环境变量
		建立执行软链接
		sudo ln -s /usr/local/idea-IU-201.6668.113/bin/idea.sh /usr/bin/idea

	5. 创建桌面快捷启动方式
		在/usr/share/applications/目录下创建一个idea.desktop文件
			[Desktop Entry]
			Type=Application
			Name=idea
			Comment=IntelliJIDEA
			Terminal=false
			Icon=/usr/local/idea-IU-201.6668.113/bin/idea.png
			Exec=/usr/local/idea-IU-201.6668.113/bin/idea.sh
       并赋予执行权限 chmod +x /usr/share/applications/idea.desktop



```

```
安装包及破解idea2019.3文件
	下载链接：https://pan.baidu.com/s/1-kE2uWEsf0Ko6vKXlLJTEw 提取码：34dq。



安装教程：
第一步：正常的安装idea，一直下一步到finish
第二步：将刚才的破解文件 （jetbrains-agent.jar） 放入安装的目录的bin目录下
可以参考我的 /usr/local/ideaIU-2019.3.4xxxxx/bin

第三步：打开idea，并选择免费试用
第四步：点击最上方的Help->Edit Custom VM Options
如果提示要创建文件，点击“yes”，没有的话忽略，这时候会打开一个文件。

第五步：在文件的最下面添加上一句话
-javaagent:自己idea的安装目录\jetbrains-agent.jar
即：-javaagent:/usr/local/ideaIU-2019.3.4xxxxx/bin/jetbrains-agent.jar

第六步：先重启idea（必须），然后点击Help->Register...

第七步：
法一：
	选择最后一个License server
	输入：http://fls.jetbrains-agent.com
	或者：http://jetbrains-license-server
	然后点击Activate

法二：
选择Activation code方式，拷贝如下代码（有点长别复制错了）：
A82DEE284F-eyJsaWNlbnNlSWQiOiJBODJERUUyODRGIiwibGljZW5zZWVOYW1lIjoiaHR0cHM6Ly96aGlsZS5pbyIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiJVbmxpbWl0ZWQgbGljZW5zZSB0aWxsIGVuZCBvZiB0aGUgY2VudHVyeS4iLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlMwIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiV1MiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJSRCIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlJDIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREMiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJEQiIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlJNIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiRE0iLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJBQyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkRQTiIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkdPIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUFMiLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJDTCIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IlBDIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlNVIiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In1dLCJoYXNoIjoiODkwNzA3MC8wIiwiZ3JhY2VQZXJpb2REYXlzIjowLCJhdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJpc0F1dG9Qcm9sb25nYXRlZCI6ZmFsc2V9-5epo90Xs7KIIBb8ckoxnB/AZQ8Ev7rFrNqwFhBAsQYsQyhvqf1FcYdmlecFWJBHSWZU9b41kvsN4bwAHT5PiznOTmfvGv1MuOzMO0VOXZlc+edepemgpt+t3GUHvfGtzWFYeKeyCk+CLA9BqUzHRTgl2uBoIMNqh5izlDmejIwUHLl39QOyzHiTYNehnVN7GW5+QUeimTr/koVUgK8xofu59Tv8rcdiwIXwTo71LcU2z2P+T3R81fwKkt34evy7kRch4NIQUQUno//Pl3V0rInm3B2oFq9YBygPUdBUbdH/KHROyohZRD8SaZJO6kUT0BNvtDPKF4mCT1saWM38jkw==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5ndaik1GD0nyTdqkZgURQZGW+RGxCdBITPXIwpjhhaD0SXGa4XSZBEBoiPdY6XV6pOfUJeyfi9dXsY4MmT0D+sKoST3rSw96xaf9FXPvOjn4prMTdj3Ji3CyQrGWeQU2nzYqFrp1QYNLAbaViHRKuJrYHI6GCvqCbJe0LQ8qqUiVMA9wG/PQwScpNmTF9Kp2Iej+Z5OUxF33zzm+vg/nYV31HLF7fJUAplI/1nM+ZG8K+AXWgYKChtknl3sW9PCQa3a3imPL9GVToUNxc0wcuTil8mqveWcSQCHYxsIaUajWLpFzoO2AhK4mfYBSStAqEjoXRTuj17mo8Q6M2SHOcwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQBonMu8oa3vmNAa4RQP8gPGlX3SQaA3WCRUAj6Zrlk8AesKV1YSkh5D2l+yUk6njysgzfr1bIR5xF8eup5xXc4/G7NtVYRSMvrd6rfQcHOyK5UFJLm+8utmyMIDrZOzLQuTsT8NxFpbCVCfV5wNRu4rChrCuArYVGaKbmp9ymkw1PU6+HoO5i2wU3ikTmRv8IRjrlSStyNzXpnPTwt7bja19ousk56r40SmlmC04GdDHErr0ei2UbjUua5kw71Qn9g02tL9fERI2sSRjQrvPbn9INwRWl5+k05mlKekbtbu2ev2woJFZK4WEXAd/GaAdeZZdumv8T2idDFL7cAirJwcrbfpawPeXr52oKTPnXfi0l5+g9Gnt/wfiXCrPElX6ycTR6iL3GC2VR4jTz6YatT4Ntz59/THOT7NJQhr6AyLkhhJCdkzE2cob/KouVp4ivV7Q3Fc6HX7eepHAAF/DpxwgOrg9smX6coXLgfp0b1RU2u/tUNID04rpNxTMueTtrT8WSskqvaJd3RH8r7cnRj6Y2hltkja82HlpDURDxDTRvv+krbwMr26SB/40BjpMUrDRCeKuiBahC0DCoU/4+ze1l94wVUhdkCfL0GpJrMSCDEK+XEurU18Hb7WT+ThXbkdl6VpFdHsRvqAnhR2g4b+Qzgidmuky5NUZVfEaZqV/g==



第八步：over，破解完成
```


```
安装包及破解idea2020.1文件

安装教程：
第一步：正常的安装idea，一直下一步到finish
第二步：将刚才的破解文件 （idea2020.1破解包/jar/jetbrains-agent.jar） 放入安装的目录的bin目录下
可以参考我的 /usr/local/idea-IU-201.6668.113/bin

第三步：打开idea，并选择免费试用
第四步：点击最上方的Help->Edit Custom VM Options
如果提示要创建文件，点击“yes”，没有的话忽略，这时候会打开一个文件。

第五步：在文件的最下面添加上一句话
-javaagent:自己idea的安装目录\jetbrains-agent.jar
即：-javaagent:/usr/local/idea-IU-201.6668.113/bin/jetbrains-agent.jar

第六步：先重启idea（必须），然后点击Help->Register...

重启动后会变为
-javaagent:/home/用户名/.jetbrains/jetbrains-agent-v3.0.0.jar

第七步：

选择Activation code方式，拷贝如下代码（有点长别复制错了）：
3AGXEJXFK9-eyJsaWNlbnNlSWQiOiIzQUdYRUpYRks5IiwibGljZW5zZWVOYW1lIjoiaHR0cHM6Ly96aGlsZS5pbyIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkFDIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9LHsiY29kZSI6IkRQTiIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJQUyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJHTyIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJETSIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJDTCIsImZhbGxiYWNrRGF0ZSI6IjIwODktMDctMDciLCJwYWlkVXBUbyI6IjIwODktMDctMDcifSx7ImNvZGUiOiJSUzAiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUkQiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUEMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUk0iLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiV1MiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREIiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiREMiLCJmYWxsYmFja0RhdGUiOiIyMDg5LTA3LTA3IiwicGFpZFVwVG8iOiIyMDg5LTA3LTA3In0seyJjb2RlIjoiUlNVIiwiZmFsbGJhY2tEYXRlIjoiMjA4OS0wNy0wNyIsInBhaWRVcFRvIjoiMjA4OS0wNy0wNyJ9XSwiaGFzaCI6IjEyNzk2ODc3LzAiLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0=-WGTHs6XpDhr+uumvbwQPOdlxWnQwgnGaL4eRnlpGKApEEkJyYvNEuPWBSrQkPmVpim/8Sab6HV04Dw3IzkJT0yTc29sPEXBf69+7y6Jv718FaJu4MWfsAk/ZGtNIUOczUQ0iGKKnSSsfQ/3UoMv0q/yJcfvj+me5Zd/gfaisCCMUaGjB/lWIPpEPzblDtVJbRexB1MALrLCEoDv3ujcPAZ7xWb54DiZwjYhQvQ+CvpNNF2jeTku7lbm5v+BoDsdeRq7YBt9ANLUKPr2DahcaZ4gctpHZXhG96IyKx232jYq9jQrFDbQMtVr3E+GsCekMEWSD//dLT+HuZdc1sAIYrw==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5ndaik1GD0nyTdqkZgURQZGW+RGxCdBITPXIwpjhhaD0SXGa4XSZBEBoiPdY6XV6pOfUJeyfi9dXsY4MmT0D+sKoST3rSw96xaf9FXPvOjn4prMTdj3Ji3CyQrGWeQU2nzYqFrp1QYNLAbaViHRKuJrYHI6GCvqCbJe0LQ8qqUiVMA9wG/PQwScpNmTF9Kp2Iej+Z5OUxF33zzm+vg/nYV31HLF7fJUAplI/1nM+ZG8K+AXWgYKChtknl3sW9PCQa3a3imPL9GVToUNxc0wcuTil8mqveWcSQCHYxsIaUajWLpFzoO2AhK4mfYBSStAqEjoXRTuj17mo8Q6M2SHOcwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQBonMu8oa3vmNAa4RQP8gPGlX3SQaA3WCRUAj6Zrlk8AesKV1YSkh5D2l+yUk6njysgzfr1bIR5xF8eup5xXc4/G7NtVYRSMvrd6rfQcHOyK5UFJLm+8utmyMIDrZOzLQuTsT8NxFpbCVCfV5wNRu4rChrCuArYVGaKbmp9ymkw1PU6+HoO5i2wU3ikTmRv8IRjrlSStyNzXpnPTwt7bja19ousk56r40SmlmC04GdDHErr0ei2UbjUua5kw71Qn9g02tL9fERI2sSRjQrvPbn9INwRWl5+k05mlKekbtbu2ev2woJFZK4WEXAd/GaAdeZZdumv8T2idDFL7cAirJwcrbfpawPeXr52oKTPnXfi0l5+g9Gnt/wfiXCrPElX6ycTR6iL3GC2VR4jTz6YatT4Ntz59/THOT7NJQhr6AyLkhhJCdkzE2cob/KouVp4ivV7Q3Fc6HX7eepHAAF/DpxwgOrg9smX6coXLgfp0b1RU2u/tUNID04rpNxTMueTtrT8WSskqvaJd3RH8r7cnRj6Y2hltkja82HlpDURDxDTRvv+krbwMr26SB/40BjpMUrDRCeKuiBahC0DCoU/4+ze1l94wVUhdkCfL0GpJrMSCDEK+XEurU18Hb7WT+ThXbkdl6VpFdHsRvqAnhR2g4b+Qzgidmuky5NUZVfEaZqV/g==

第八步：over，破解完成

```

> 参考资料

+ [IntelliJ IDEA安装破解教程（附安装包2019.3 版+破解文件+激活方法)](https://www.cnblogs.com/chaogu94/p/12175819.html)
+ [2019版 IntelliJ IDEA破解，2020/3/11更新](https://www.jianshu.com/p/c6192e540d45)

+ [idea 2020年最新破解补丁（亲测有效）](http://errornoerror.com/question/1585959663051139/) 



```
	遇到问题：
	
	1. dwm桌面下 打开时只有主题的纯色
	
		解决办法：
			sudo pacman -S wmname
			添加下面的代码到~/.xinitrc

			export _JAVA_AWT_WM_NONREPARENTING=1 
			export AWT_TOOLKIT=MToolkit 
			wmname LG3D

		之后reboot或者source ~/.xinitrc

```

> 参考资料

+ [Linux系统安装 IntelliJ IDEA](https://www.cnblogs.com/418836844qqcom/p/10722197.html)

+ [求问:archlinux下使用dwm窗口,运行idea的时候,设置页面打开项目过程都是正常的,项目打开之后就整个窗口就变成了纯主题色,像是黑屏那种](https://blog.csdn.net/weixin_44513641/article/details/104738842)
+ [idea-dwm](https://blog.csdn.net/u010563350/article/details/104948256)

+ [在linux中添加应用程序到applications列表](https://blog.csdn.net/huahuajjh/article/details/63263485)

### 安裝maven

```
1. 去官网下载maven的压缩包《maven官网[http://maven.apache.org/download.cgi]》

点击左侧下载(Download)

下载完成后，上传到linux上

2. 解压maven的tar文件

tar -zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local

3. 配置环境变量

vim编辑文件

vim /etc/profile
输入i，进入编辑模式，profile的文件末尾，新增配置

export MAVEN_HOME=/usr/local/apache-maven-3.6.3
export PATH=${PATH}:${MAVEN_HOME}/bin

按返回键（ESC），进入一般模式，输入冒号，然后输入wq，保存退出

4. 刷新配置文件
source /etc/profile

5. 测试
mvn -v

```

> 参考资料
- [linux下安装maven](https://blog.csdn.net/qq_28410283/article/details/81837151)


### 文件共享、投屏： minidlna

```
多媒体共享服务器，类似于FTP，支持DLNA的客户端都可以看视频，听音乐，处于同一局域网就可以了

安装  sudo pacman -S minidlna


配置：

编辑 /etc/minidlna.conf 来设定分享目录:
sudo  vim /etc/minidlna.conf

media_dir=/srv/media
第一行会把 /srv/media 底下所有的媒体文件(照片, 影片, 音乐)分享出去. 如果想要限制媒体的种类, 可以在目录前加上 V(影片), A(声音)或 P(照片)来指定种类:
如:
在minidlna.conf文件里找到
  + "A" for audio  (eg. media_dir=A,/home/jmaggard/Music)
  + "V" for video  (eg. media_dir=V,/home/jmaggard/Videos)
  + "P" for images (eg. media_dir=P,/home/jmaggard/Pictures)

的部分。这是设置存放多媒体文件的文件夹（严格地说是设置想要MiniDLNA扫描的文件夹）的地方。“A”是音乐文件。“V”是视频文件。“P"是图片文件。
比如我的设置是：
media_dir=A,/home/mike/Public/Playlists （我存放playlist的地方）
media_dir=A,/home/mike/Public/Music （我存放音乐文件的地方）
media_dir=V,/home/mike/Public/Video （我存放视频文件的地方）
media_dir=P,/home/mike/Public/Pictures （我存放图片的地方）

MiniDLNA是要通过提示符下的命令行来启动的。
- 启动     systemctl start minidlna
- 停止     systemctl stop minidlna
- 查看状态 systemctl status minidlna

另外也别忘了指定 port 以及 server 的名称(也可以用默认的)

port=8200
model_name=My DLNA Server

然后更新 cache

sudo  minidlnad -R


```

- [MiniDLNA服务器的设置和启动_Mike20110821_新浪博客](http://blog.sina.com.cn/s/blog_8a06d57b01012c92.html)

### 系统间进行文件、打印机共享：Samba



### 目录搜索工具：find

### 目录搜索工具: fd

---
```
    安装
        sudo pacman -S fd
    使用
       例子：
       fd 关键词或省缺为空 --full-path '目录地址' -t d或f或l
      
       fd xiexie1993  -p '/' -t f   "查找根目录下的含有文件目录地址中含有xiexie1993的
       fd xiexie1993  -p '/' -t d   "查找根目录下的含有文件夹地址中含有xiexie1993的
       
```

### 安装texlive (LaTex)

---
```
	sudo pacman -S texlive-core texlive-langchinese texlive-latexextra

	安装texlive-localmanager-git
	aurman texlive-localmanager-git

	使用localmanager安装缺少部分
	方法 使用tllocalmgr 进入命令行，之后缺什么安装什么
	命令例子： 
		tllocalmgr
		install 包名
	
	还可再装gui界面编辑软件
	sudo pacman -S  texstudio
```

> 参考资料
+[archlinux 安装latex使用中文](http://blog.csdn.net/qq_29343201/article/details/78481114)

### 文本搜索工具： ack

### 文本搜索工具：ag

### 文本搜索工具：ripgrep (rg)

### 社交通讯： qq

```
    yay -S qq-linux

```

### 社交微信： electronic-wechat

```
    yay -S electronic-wechat
```

### 照片识别文字:textshot

```
yay -S textshot-git
注： 目前只有识别英文
```

- [python开源照片识别文字](https://github.com/ianzhao05/textshot)

### 命令行里查看图片:fim  viu 

+ 方案一: fim
	+ 安装
	```
	yay -S fim
	```
	+ 使用
	```
	使用命令显示带有“自动缩放”选项（ -a ）的图像：
		$ fim -a dog.jpg
	FIM 支持一次性打开多个文件，例如，当前目录有很多个 .jpg 文件，你可以使用通配符 * 来打开这些文件：
		$ fim -a * .jpg
	或者，要打开目录中的所有图像，例如 Pictures ：
		$ fim Pictures/
	我们还可以递归地打开文件夹及其子文件夹中的图像，然后对列表进行如下排序：
		$ fim -R Pictures/ --sort
	要以 ASCII 格式呈现图像，可以使用 -t 选项：
		$ fim -t dog.jpg
	退出 FIM 请按 ESC 或者 q 。
	```
	+ 用于控制 FIM 中图像的常用快捷键：
	```
	PageUp / Down：上一个图像/下一个图像
	+/-：放大/缩小
	a：自动缩放
	w：合适宽度
	h：合适身高
	j / k：向下平移/向上平移
	f / m：翻转/镜面反射
	r / R：旋转（顺时针和逆时针）
	ESC / q：退出
	```

+ 方案二： viu
	+ 安装
	```
	yay -S viu
	```

+ 方案三：imagemagick+Lsix
> 确保你的终端支持 Sixel 格式
	+ 安装
	```
	sudo pacman -S imagemagick
	wget https://github.com/hackerb9/lsix/archive/master.zip
	unzip lsix-master.zip
	sudo cp lsix-master/lsix /usr/local/bin/
	sudo chmod +x /usr/local/bin/lsix
	```
	> lsix 本身其实就是个 BASH 脚本，所以无需进行安装，只需将它下载下来，并移动到 $PATH 环境变量里。

	> Xterm 上以 vt340 仿真模式来开发了 lsix ，但 Xterm 并不默认支持 Sixel 。启动支持 Sixel 的方式如下：
	```
	xterm -ti vt340
	```
	```
	Xterm 默认开启 Sixel ，需要修改它的 .Xresources 文件（如果没有这个文件，直接创建一个即可）：
	vim .Xresources
	在文件里添加这么一句：
	xterm*decTerminalID    :   vt340
	```
> 参考资料
- [什么？Linux 终端也可以用来看女神照片？](https://mp.weixin.qq.com/s?__biz=MzU3NTgyODQ1Nw==&mid=2247485577&idx=1&sn=481418e5efdfd0b06095c6e07d890628&chksm=fd1c700fca6bf9194a6657ab06562f4f0890967d7491880804d0c9aa08a983ee1ab14a4daf34&token=1994080286&lang=zh_CN#rd)
- [活久见！Linux命令行居然也可以用来查看图像？](https://www.cnblogs.com/yychuyu/archive/2020/04/13/12690826.html)

### 图片查看器

+ feh
	```
	命令行启动 Feh：只将其指向图像或者包含图像的文件夹之后就行了。Feh 会快速加载，你可以通过鼠标点击或使用键盘上的向左和向右箭头键滚动图像。不能更简单了。
	Feh 可能很轻量级，但它提供了一些选项。例如，你可以控制 Feh 的窗口是否具有边框，设置要查看的图像的最小和最大尺寸，并告诉 Feh 你想要从文件夹中的哪个图像开始浏览。

	安装
		sudo pacman -S feh
	```

+ Ristretto 
	```
	打开包含图像的文件夹，单击左侧的缩略图之一，然后单击窗口顶部的导航键浏览图像。Ristretto 甚至有幻灯片功能。
	Ristretto 也可以做更多的事情。你可以使用它来保存你正在浏览的图像的副本，将该图像设置为桌面壁纸，甚至在另一个应用程序中打开它，例如，当你需要修改一下的时候。

	安装
		sudo pacman -S ristretto
	```

+ Nomacs
	```
	可以查看和编辑图像的元数据，向图像添加注释，并进行一些基本的编辑，包括裁剪、调整大小、并将图像转换为灰度。Nomacs 甚至可以截图。
	一个有趣的功能是你可以在桌面上运行程序的两个实例，并在这些实例之间同步图像

	安装
		sudo pacman -S nomacs
	```

+ Mirage
	```
	Mirage有点平常，没什么特色，但它做着和其他优秀图片浏览器一样的事：打开图像，将它们缩放到窗口的宽度，并且可以使用键盘滚动浏览图像。它甚至可以使用幻灯片。
	不过，Mirage 将让需要更多功能的人感到惊喜。除了其核心功能，Mirage 还可以调整图像大小和裁剪图像、截取屏幕截图、重命名图像，甚至生成文件夹中图像的 150 像素宽的缩略图。
	Mirage 还可以显示 SVG 文件

	安装
	 yay -S mirage
	```

+ gThumb
	```
	查看/管理界面和编辑工具（裁剪、缩放、颜色编辑等等）将会给你留下很深的印象。
	你甚至可以为图像添加评论或清除它的 EXIF 信息。它使得你可以方便地找到重复的图像或转码图像。

	安装
	 yay -S gthumb
	```

> 参考资料
- [4 个 Linux 桌面上的轻量级图像浏览器](https://linux.cn/article-8776-1.html)
- [Linux 下最棒的 11 个图片查看器](https://www.jianshu.com/p/41f021f47eec)
- [适用于 Linux 系统的 11 款图像查看器](https://blog.csdn.net/ybhuangfugui/article/details/106184314)

### Ulauncher :程序启动

### stacer ：

### timeshift ： 系统备份

### etcher u盘硬盘刻录

### cayden live 视频编辑

### rofi 窗口导航/程序搜索切换

+ 安装
```
sudo pacman -S rofi
```

+ 配置
```
配置主题文件位置： ~/.config/rofi
配置基本例子：
configuration {
 modi: "window,drun,ssh,combi";
 theme: "solarized";
 font: "hack 10";
 combi-modi: "window,drun,ssh";
 }


主题预览查看选择命令
rofi-theme-selector
```

+ 使用
```
启动配置,是以命令行参数的形式添加的配置,命令如下
rofi -show drun -theme fancy
或
rofi -show drun -theme gaara-theme
或
rofi -show window -theme gaara-theme
或
rofi -show ssh -theme gaara-theme

参数说明：
drun      桌面启动目录（/usr/share/applications）
ssh       记录的ssh
window   当前打开窗口
run       当前运行任务

```

+ 问题处理记录
```
报错： Failed to set locale, 

查询可知locale是用来设置语言环境的，故此需要查看并正确设置locale

解决方案：

echo ”export LC_ALL=en_US.UTF-8“  >>  /etc/profile 

```

## 软件收集记录 



hexChat 世界通用IRC聊天客户端

spotify 一个正版流媒体音乐服务平台客户端

calibre E-book 电子书管理器

开源笔记本软件 Joplin joplin-desktop

Nitrogen 一个快速和轻量级的桌面背景浏览器和设置器的X窗口

> 参考资料

> + [20个堪称神器的Linux命令行软件](http://www.zijin.net/news/tech/1480185.html)


## 问题

### 执行sudo pacman -Syu 时出现错误

到此处查看该处会及时更新升级时遇到问题的解决办法 [Artix Linux - Home](https://artixlinux.org/)

### 开机 出现Failed to start Load Kernel Modules

开机出现加载内核失败，但是能正常开机，电脑一直好好的，莫名其妙的出现了这个问题，经过检查原因是安装VirtualBox的时候安装了 virtualbox-host-dkms ，导致开机需要加载kernel
解决方法安装linux-headers:

 sudo pacman -S linux-headers

- [开机 出现Failed to start Load Kernel Modules（Archlinux)](https://blog.csdn.net/r8l8q8/article/details/76226442s)

```
systemctl --state=failed
sudo systemctl status systemd-modules-load.service
```

+ PID=117  Failed to find module 'vboxdrv'
```
sudo journalctl -b _PID=117
sudo pacman -S virtualbox-host-dkms
sudo dkms install vboxhost/4.3.10
sudo systemctl enable dkms.service
sudo vim /etc/modules-load.d/virtualbox.conf
```

+ PID=324  Failed to find module 'vfs_monitors'： Invlid argument


- [修复："Failed to start Load Kernel Modules"](https://www.cnblogs.com/zhangshaojian/p/3702247.html)
- [ [SOLVED] Failed to insert 'vfs_monitor': Invalid argument](https://bbs.archlinux.org/viewtopic.php?id=238115)
- [service启动失败问题排查](https://blog.csdn.net/wenjianmuran/article/details/90633403)
- [[FAILED] Failed to start Load Kernel Modules错误处理](https://blog.csdn.net/yuxipro/article/details/109102059)


### dwm和st打不开

原因分析：未安装或程序所在目录（/usr/local/bin）未加入系统环境变量中
	1. 打开/etc/profile查看，和执行 echo $PATH 修改和查看当前系统环境变量配置

### sddm 启动失败

```
修改配置文件：
$ sudo nano /etc/mkinitcpio.conf
更改MODULES=()，因为我是A卡用户，所以改为MODULES=("radeon")，intel核显为i915，N卡为nouveau
最后重新生成一下内核：
$ sudo mkinitcpio -p linux
重启就能正常进入 sddm 了。
```

> 参考资料
- [SDDM启动黑屏](https://www.jianshu.com/p/694cd939e80e)

### 字体管理工具: font-manager

+ 安装：
```
yay -S font-manager
```

### tty终端：中文字符方块

```
curl https://www.linux.zone/files/en > /usr/bin/en && chmod 755 /usr/bin/en

以后在tty终端里只需要在命令前加上en（比如en mc，en nano），就会显示英文的界面，不会显示tty不能识别的中文字符了。

en脚本的内容就两行，也就是一句代码。你可以将其拷贝到新建文件里，保存为/usr/bin/en，然后设置其属性为755即可：

#!/bin/sh

export LANG=en_US.UTF-8 && "$@"

相反，如果想切回到中文语言，以root用户身份运行如下一条命令：

curl https://www.linux.zone/files/zh > /usr/bin/zh && chmod 755 /usr/bin/zh

以后在tty终端字符界面里只需要在命令前加上zh（比如zh mc，zh nano） ，就会将当前语言临时切换到中文。不过系统语言如果设置为中文，一般都会显示中文的。

zh脚本的内容：

#!/bin/sh

export LANG=zh_CN.UTF-8 && "$@"
```

> 参考资料
- [Linux tty字符界面显示方块乱码？一条命令可以解决](https://linux.zone/2337)
- [linux 终端界面显示 中文乱码或方块](https://blog.csdn.net/whatday/article/details/106189385)


## 开机启动时报错无法正常启动

+ you are in emergency mode解决办法
```
造成原因：
可能是挂载异常导致（另外的盘重装系统，uuid变量导致原来自定义的挂载点成了错误的了）

解决办法：
1、在这个界面直接输入密码，然后回车
2、vim /etc/fstab
原来之前在这里增加过一行，现在删除，保存退出

详细解决过程：
1）先查看日志，
journalctl -xb

2）使用查找命令,看看哪个磁盘出错。
/ fsck failed

使用n可以往下查找下一个相关字段。
比如我的是
fsck failed with exit status 4
再往下看几行，找到有uuid编号的那一行，,记住那个编号。比如我的是
file system check on /dev/disk/by-uuid/06f26d84-cc4c-4abf-9fbc-6a16f56024f7

3）
输入:q 回车 退出journal日志
输入 vi /etc/fstab查看自己的磁盘编号。如果有除了/、/boot、swap、/home之外的磁盘，就使用dd删除那一行。
接下来找到uuid编号一样的那个sda盘。

4）
输入:q 回车 退出fstab
使用如下命令：
umount /dev/sdax      // x是你自己的磁盘编号
fsck -y /dev/sdax
reboot    
```

> 参考资料
- [you are in emergency mode解决办法](https://blog.csdn.net/weixin_40689871/article/details/108451501)


### 更新后 append_path: command not found

+ 原因：
更新系统的时候，由于我之前修改了/etc/profile文件，导致/etc/profile不能直接升级，就生成了一个/etc/profile.pacnew文件，然后让你手动修改


+ 解决办法：
只需要合并/etc/profile和/etc/profile.pacnew文件即可

```
sudo mv /etc/profile /etc/profile.old
sudo mv /etc/profile.pacnew /etc/profile

然后在把之前修改的东西加到/etc/profile中去就可以了

source /etc/profile 
```

> 参考资料
- [append_path: command not found](https://blog.csdn.net/CHAOS_ORDER/article/details/110947913)


## android容器：Anbox

```
# 安装linux-zen 内核自带 Anbox 需要的模块。这可能是最简单的方式，因为您不必编译内核并且版本会定期更新。
yay -S linux-zen


# 挂载 binderfs
Note: linux-zen 内核必须挂载 binderfs。

vim /etc/tmpfiles.d/anbox.conf
在 /etc/tmpfiles.d/ 创建一个包含以下内容的文件：

d! /dev/binderfs 0755 root root

启动时挂载它，只需要在 /etc/fstab 中添加下面这一行。
none                         /dev/binderfs binder   nofail  0      0


# 安装 Android 镜像


# 安装 Anbox
yay -S anbox-git

# 启动anbox-container-manager.service服务
systemctl start anbox-container-manager.service

# 开机启动anbox-container-manager.service服务
systemctl enable anbox-container-manager.service

# 网络配置
（如果您正在使用 NetworkManager ，则可以使用它来配置网络。
执行以下命令来创建 bridge connection：）
nmcli con add type bridge ifname anbox0 -- connection.id anbox-net ipv4.method shared ipv4.addresses 192.168.250.1/24

```

```
查看 app列表
adb shell pm list packages

安装 app
adb install /path/to/app.apk

卸载 app
adb uninstall app.name
```


> 参考资料
- [Anbox (简体中文)](https://wiki.archlinux.org/title/Anbox_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))


### 动态壁纸


```
Manjaro/Arch安装
    yay -S komorebi
```

> 参考资料
- [archlinux动态壁纸展示及简单安装制作](https://www.bilibili.com/video/av969471706/)
- [Komorebi--Linux下动态壁纸](https://www.jianshu.com/p/e985d374c50b)


###  系统清理

```
sudo pacman -R $(pacman -Qdtq)          # 清理系统中无用的依赖包
sudo pacman -Scc                         # 清理缓存，看路径像是之前下载的安装包

sudo du -sh /var/cache/pacman/pkg/     # 查看pacm缓存的安装文件大小

sudo journalctl --disk-usage                # 查看日志大小
sudo journalctl --vacuum-time=5d            # 超过5天的自动删除
sudo journalctl --vacuum-size=500M          # 超过500M的自动删除
sudo rm /var/lib/systemd/coredump/*         # 崩溃日志，文件不多，也不大，删不删随你
```

> 参考资料
- [Archlinux 系统清理](https://blog.csdn.net/Amy_wangxs/article/details/108950616)
