# Arch LInux 的kvm 安装

+ 检测宿主机cpu是否支持虚拟化，如果flags里有vmx 或者svm就说明支持VT

```bash
$ grep -E "(vmx|svm)" --color=always /proc/cpuinfo
```

+ 检查内核的KVM和VirtIO模块是否可用

```bash
$ zgrep KVM /proc/config.gz
$ zgrep VIRTIO /proc/config.gz 
```

+ 查看内核模块是否装载

```bash
$ lsmod | grep kvm 
$ lsmod | grep virtio
```

+ + 手动加载内核模块

```bash
$ sudo modprobe virtio
```

当前用户加入组kvm

```bash
$ sudo usermod -a -G kvm username /*username改为你的用户名*/
```

+ 安装qemu以及图形化客户端

```bash
$ sudo pacman -S qemu
$ sudo pacman -S libvirt virt-manager
```

+ 连接网络需要安装

```bash
$ sudo pacman -S ebtables dnsmasq bridge-utils openbsd-netcat
```

+ 设置授权

```bash
$ sudo vim /etc/polkit-1/rules.d/50-libvirt.rules
```

```shell
/* Allow users in kvm group to manage the libvirt daemon without authentication */
polkit.addRule(function(action, subject) {
    if (action.id == "org.libvirt.unix.manage" &&
        subject.isInGroup("kvm")) {
            return polkit.Result.YES;
    }
});
```

+ 启动服务

```bash
$ sudo systemctl enable libvirtd
$ sudo systemctl start libvirtd
$ sudo systemctl start virtlogd
```

启动virt-manager就可以使用kvm了