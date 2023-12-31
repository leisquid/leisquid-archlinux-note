# leisquid 的 Arch Linux Installation Guide

（以 UEFI 环境为例）

本文采用 **CC BY-NC-SA 4.0** 协议授权

## 1. 安装前的准备（略）

## 2. 启动到 Live 环境

### (1) 改变键盘布局

列出所有的键盘布局：

```sh
ls /usr/share/kbd/keymaps/**/*.map.gz
```

添加布局，如德语：

```sh
loadkeys de-latin1
```

### (2) 验证引导模式

```sh
ls /sys/firmware/efi/efivars
```

如果列出目录且未报错，则系统以 UEFI 模式引导，否则可能是以 BIOS 或 CSM 模式引导。

### (3) 连网

如果是有线网且支持 DHCP：

```sh
dhcpcd
```

如果是无线网：

```sh
iwctl
device list
```

会列出无线网卡名称，如 `wlan0`，下面应替换成自己的网卡名：

```sh
station wlan0 scan
station wlan0 gei-networks  # 列出无线网名单
station wlan connect your_ssid_name
```

如果有密码，输入后按 Return。

```sh
quit
```

### (4) 更新系统时间

```sh
timedatectl set-ntp true
```

### (5) 磁盘分区

此处略。可以用可视化工具 cfdisk 处理。不会就去翻在线 Arch Linux 维基。

### (6) 挂载分区

```sh
mount /dev/root_partition /mnt
swapon /dev/swap_partition
```

对于 UEFI 系统，还需要挂载 EFI 系统分区：

```sh
mount /dev/efi_system_partition /mnt/boot
```

### (7) 更改镜像源

镜像源的文件位于：`/etc/pacman.d/mirrorlist`。

可以使用清华 TUNA 源、北京外国语大学源或者中科大源：

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrros.bfsu.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```

也可以用以下命令智能匹配最快的源，以中国为例：

```sh
reflector -c China -a 10 --sort rate --save /etc/pacman.d/mirrorlist
```

### (8) 安装必需的软件包

```sh
pacstrap /mnt base base-devel linux linux-firmware dhcpcd
```

其中内核可以替换成：

| 内核名 | 解释 |
| ---- | ---- |
| linux-hardened | 加固内核 |
| linux-lts | 长期支持的内核 |
| linux-zen | 最适合日用的内核 |

## 3. 基本安装后配置

### (1) 配置 fstab

```sh
genfstab -L /mnt >> /mnt/etc/fstab
```

### (2) chroot

```sh
arch-chroot /mnt
```

### (3) 设置时区，以上海为例

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### (4) 提前安装必要软件包

```sh
pacman -S # 这里跟上您想安装的包，以空格隔开
```

建议可以安装以下包：

| 包名 | 用途 |
| ---- | ---- |
| dialog | 在 shell 脚本显示对话框的工具 |
| wpa_supplicant | 无线网支持 |
| ntfs-3g | NTFS 驱动及工具 |
| networkmanager | 网络连接管理及用户工具 |
| netctl | 网络管理 |
| vim | 文本编辑器 |
| nano | 文本编辑器，比 vim 易用 |

### (5) 设置 locale

打开 `/etc/locale.gen`，反注释 `zh_CN.UTF-8`、`en_US.UTF-8`，然后执行 `locale-gen`。

### (6) 配置系统语言

打开（创建）`/etc/locale.conf`，添加内容：`LANG=en_US.UTF-8`。

### (7) 设置主机名

创建 `/etc/hostname`，输入主机名。

### (8) 配置 `/etc/hosts`

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   your_hostname.localdomain   your_hostname
```

其中 `your_hostname` 替换为您在上一步设置的主机名，如果您的主机所在的网络内定义了域名，可将 `localdomain` 替换为您的域名。

### (9) 设置 root 密码

执行 `passwd`。

### (10) 启用微码更新

AMD 处理器安装 amd-ucode；Intel 处理器安装 intel-ucode。

### (11) 安装引导程序，以 grub2 为例

安装 grub 和 efibootmgr。

```sh
pacman -S grub efibootmgr
```

部署 grub：

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

生成配置文件：

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

中文环境下执行上面的指令可以生成含中文配置的配置文件。

### (12) 重启

```sh
exit    # 或者按 Control+D
umount /mnt/boot
umount /mnt
reboot
```

## 4. 安装后配置

完成上面的重启步骤后，会进入到新安装的系统里。

### (1) 连网

有线网用 dhcpcd，无线网用 wifi-menu。

### (2) 新建一个普通用户

在 root 环境下操作有风险，建议日常使用时以普通用户登录并使用。

新建一个用户组并将其加入到组 wheel，这样可以使用 sudo 的一些功能。

```sh
useradd -m -G wheel your_username   # 将 username 替换为您想起的用户名
passwd your_username                # 给这个新用户设置一个密码
```

### (3) 配置 sudo

如果没安装，就先安装 sudo。

```sh
pacman -S sudo
```

虽然一般修改 sudo 配置文件需要用到 vi，但是可以将任一编辑器改为用于修改 sudo 配置文件的编辑器。可以临时设置环境变量。比如：

```sh
EDITOR=nano visudo
```

找到 `# %wheel ALL = (ALL) ALL` 这一行并反注释。保存配置文件后重启，以新用户登录并连网。

## 5. 安装图形界面

### (1) 安装显卡驱动

| Brand | Type | Driver | OpenGL |
| :----: | :----: | :----: | :----: |
| AMD / ATI | open source | xf86-video-amdgpu | mesa |
| ↑ | ↑ | xf86-video-ati | ↑ |
| ↑ | proprietary | catalyst (AUR) | catalyst-libgl (AUR) |
| Intel | open source | xf86-video-intel | mesa |
| Nvidia | open source | xf86-video-nouveau | mesa |
| ↑ | propriety | nvidia | nvdida-utils |
| ↑ | ↑ | nvidia-340xx | nvidia-340xx-utils |
| ↑ | ↑ | nvidia-304xx | nvidia-304xx-utils |

### (2) 安装 xorg

```sh
sudo pacman -S xorg
```

### (3) 安装桌面环境

如安装 Xfce，则安装 xfce4、xfce-goodies；
如安装 KDE Plasma，则安装 plasma、kde-applications。

```sh
# 以下命令根据您对桌面环境的需求二选一。
sudo pacman -S xfce4 xfce-goodies
sudo pacman -S plasma kde-applications
```

### (4) 安装桌面管理器，以 sddm 为例

安装 sddm，并配置开机启动：

```sh
sudo pacman -S sddm
sudo systemctl enable sddm
```

### (5) 提前配置网络

用更好一点的工具接管网络配置。

```sh
sudo systemctl disable netctl
sudo systemctl enable NetworkManager
```

确保安装了 network-manager-applet。

### (6) 重启

重启后进入图形环境。您在登录后可根据需要对系统和图形环境进行进一步配置。
