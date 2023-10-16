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

可以使用清华 TUNA 源或者中科大源：

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
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
genfstab -L /mnt >> /mnt/stc/fstab
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
pacman -S # 这里跟上你想安装的包，以空格隔开
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
