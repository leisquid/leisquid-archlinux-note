# leisquid 的 Arch Linux Installation Guide

（以 UEFI 环境为例）

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
