# 使用 Arch 时可能会出现的各种问题

本文采用 **CC BY-NC-SA 4.0** 协议授权

## 将硬件时间设置为 localtime

Arch Linux 默认会将硬件时间视为 UTC 时间。如果电脑安装了 Windows 等其他系统，会导致系统时间混乱。有一种办法是让 Arch 做出妥协，将硬件时间作为本地时间。

以下命令需要 root 权限执行。

```sh
timadatectl set-local-rtc 1
```

## git 跳过证书检测

```sh
git config --global http.sslVerify false
```

## Plasma 语言配置问题

删除 `~/.config/plasma-localerc`。

## Plasma 启动器不更新

(1) 删除 `~/.config/Trolltech.conf`。

(2) 执行 `kbuildsycoca5 --noincremental`。

## 启用内核快捷键

启用后，当系统出现问题（还有响应且内核没有 panic）时，可以使用一组快捷键，紧急保存您的数据并重启。

启用方法有以下几种，如前一种方法不管用，可使用下一种方法。

(1) 使用 root 权限执行：

```sh
echo "1" > /proc/sys/kernel/sysrq
```

(2) 使用 root 权限执行：

```sh
echo "kernel.sysrl = 1" > /etc/sysctl.d/99-sysctl.conf
```

(3) 添加内核启动参数：`sysrq_always_enabled=1`

在哪里加呢？可以加在 GRUB 配置文件里。GRUB 配置文件在 `/etc/default/grub`，编辑这个文件，在 `GRUB_CMDLINE_LINUX_DEFAULT` 一行添加 `sysrq_always_enabled=1` 并保存，然后以 root 权限执行：

```sh
update-grub
```

或者：

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

重启 Linux 内核后生效。

## 如何使用内核快捷键

| 组合键 | 作用 | 含义 |
| ---- | ---- | ---- |
| Alt+SysRq+**R** | Unraw | 从 X （图形服务器）收回键盘控制 |
| Alt+SysRq+**E** | Terminate | 向所有进程发送 SIGTERM 信号 |
| Alt+SysRq+**I** | Kill | 向所有进程发送 SIGKILL 信号 |
| Alt+SysRq+**S** | Sync | 将待写入的数据写入硬盘 |
| Alt+SysRq+**U** | Unmount | 卸载所有以挂载的硬盘并以只读方式重新挂载 |
| Alt+SysRq+**B** | Reboot | 重启 |

可以这样记忆：***R**eboot **E**ven **I**f **S**ystem **U**tterly **B**roken.*

## Pacman 智能匹配最快的源

以 root 权限执行：

```sh
reflector -c China -a 10 --sort rate --save /etc/pacman.d/mirrorlist
```

## Pacman 删除无用依赖

以 root 权限执行：

```sh
pacman -Rns $(pacman -Qdtq)
```

## 字体的全局配置

全局配置文件位于 `/etc/fonts/local.conf`，在图形界面下对所有用户有效。

参见 [fonts-local.conf](fonts-local.conf)。

## GRUB 配置

GRUB 配置文件在 `/etc/default/grub`。编辑文件后需要以 root 权限执行 `update-grub` 或 `grub-mkconfig -o /boot/grub/grub.cfg` 使配置生效。

### (1) 启用高亮上一次启动的系统

反注释 `GRUB_SAVEDEFAULT=true`，并将 `GRUB_DEFAULT` 设为 `saved`。

### (2) 启用 os-prober 以探测其他操作系统

反注释 `GRUB_DISABLE_OS_PROBER=false`。
