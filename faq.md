# 使用 Arch 时可能会出现的各种问题

本文采用 **CC BY-NC-SA 4.0** 协议授权

## 前面

如无特殊说明，“以 root 权限执行” 指的是通过执行 `su` 登录到 root 帐户执行，或者在执行的命令前加上 `sudo ` 临时提权执行。使用 sudo 前请确保您登录的帐户可以使用 sudo。

## 将硬件时间设置为 localtime

Arch Linux 默认会将硬件时间视为 UTC 时间。如果电脑安装了 Windows 等其他系统，会导致系统时间混乱。有一种办法是让 Arch 做出妥协，将硬件时间作为本地时间。

以 root 权限执行：

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

(1) 以 root 权限执行：

```sh
echo "1" > /proc/sys/kernel/sysrq
```

(2) 以 root 权限执行：

```sh
echo "kernel.sysrq = 1" > /etc/sysctl.d/99-sysctl.conf
```

(3) 添加内核启动参数：`sysrq_always_enabled=1`

在哪里加呢？如果您使用了 GRUB 作为引导管理器，那么可以加在 GRUB 配置文件里，并随着 GRUB 配置文件的更新写入到内核启动参数。GRUB 配置文件在 `/etc/default/grub`，编辑这个文件，在 `GRUB_CMDLINE_LINUX_DEFAULT` 一行添加 `sysrq_always_enabled=1` 并保存，然后以 root 权限执行：

```sh
update-grub
```

或者：

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

重新引导后生效。

## 如何使用内核快捷键让系统安全地强制重启

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

其中各选项的作用如下：

```sh
pacman --remove --nosave --recursive $(pacman --query --deps --unrequired --quiet)
```

## 字体的全局配置

全局配置文件位于 `/etc/fonts/local.conf`，使用 XML 语言，在图形界面下对所有用户有效。参见 [fonts-local.conf](./fonts-local.conf)。

另外近期发现，实际上系统会对字体进行匹配度的评测，有些字体即使用 `<alias>` `<prefer>` 标签包裹，也可能会因为评测后名称和特征不如其他字体的匹配度更高而被评为“弱匹配”候选导致最终匹配失败。可以使用 `<match>` `<test>` `<edit>` 等标签进行强匹配绑定。这部分代码要写在 `<fontconfig>` 标签内。

下面的示例代码是将 Source Code Pro 这款等宽字体与字体名 “monospace” 进行强（制）匹配，并使用简体中文字体 “思源黑体” 作为后备字体：

```xml
<match target="pattern">
    <test name="family">
        <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
        <string>Source Code Pro</string>
        <string>Source Han Sans SC</string>
        <string>Noto Sans CJK SC</string>
    </edit>
</match>
```

## GRUB 配置

GRUB 配置文件在 `/etc/default/grub`。编辑文件后需要以 root 权限执行 `update-grub` 或 `grub-mkconfig -o /boot/grub/grub.cfg` 使配置生效。

### (1) 启用高亮上一次启动的系统

反注释 `GRUB_SAVEDEFAULT=true`，并将 `GRUB_DEFAULT` 设为 `saved`。

### (2) 启用 os-prober 以探测其他操作系统

反注释 `GRUB_DISABLE_OS_PROBER=false`。

## KDE Plasma 升级故障

可通过尝试清理缓存解决。

```sh
rm -rf ~/.cache/*
```

## 由于 Windows 休眠，NTFS 分区不能被读写

以 root 权限执行：

```sh
umount /mount_point # 改成有问题的分区挂载点
ntfs-3g -o remove_hiberfile /dev/partition /mount_point # 分别改成有问题的分区和有问题的挂载点
```

## Plasma 不尊重区域/语言设置

(1) 删除 `~/.config/plasma-localerc` 后重新登录；

(2) 手动编辑 `~/.config/plasma-localerc`：

```
[Formats]
LANG=zh_CN.UTF-8

[Translations]
LANGUAGE=zh_CN:en_US
```

## pip 换源

(1) 以下是可用的源：

```
https://pypi.tuna.tsinghua.edu.cn/simple/
https://mirrors.aliyun.com/pypi/simple/
https://pypi.mirrors.ustc.edu.cn/simple/
```

(2) 临时换源

```sh
pip install packages_to_be_installed -i mirror_url
```

(3) 全局换源/换回源

```sh
pip config set global.index-url mirror_url
pip config unset global.index-url mirror_url
```

## Mplayer 播放 CD 时声音断断续续

需要 `-cache` 选项让软件提前缓冲。

```sh
mplayer cdda://:1 -cache 1024
```

其中 `:1` 是为了让光驱降速以稳定旋转并减少噪声。

## Plasma 与打印服务问题

安装 cups 并配置服务启动和开机自启。

## Pacman 的 key 问题

以 root 权限执行：

```sh
pacman-key --init
pacman-key --populate
```

## 在 Linux 中使用 VLC 播放 Blu-ray

(1) 安装 libbluray 和 libaacs。

(2) 从 http://fvonline-db.bplaced.net 下载需要的 KEYDB.cfg 并复制到 `~/.config/aacs/`。建议建立一个同名但是全大写/小写的文件名的软链。

(3) （可选）复制 https://forum.doom9.org/showpost.php?p=1883655&postcount=3 提供的 PK and Host K/C data 到 KEYDB.cfg 的头部。

(4) 挂载光盘，以 root 权限执行：

```sh
mount /dev/sr0 /mnt/bluray  # 块设备和挂载点根据自己的实际情况修改
```

(5) 查询是否可以解码：

```sh
bd_info /dev/sr0 | grep "AACS handled"  # 块设备根据自己的实际情况修改
```

如果输出有 `yes`，说明可以解码

(6) 如果到此还是不能解码，那么说明光盘内容也有可能是 BD+ 加密的，这时还需要安装 libbdplus (AUR)；如果还是不行，那就只能使用 MakeMKV (AUR) 或 DVDFab (under Wine) 这样的商业方案了。

(7) 如果在读取 RTSP 流、DVB-T 流或 Blu-rays 时，VLC 似乎始终在缓冲而且没有报错，则需要安装 aribb24。

其他软件如 MPlayer、Xine 也能播放 Blu-ray，但还是 VLC 的综合体验更好。

其他问题，参见：<https://wiki.archlinux.org/title/Blu-ray#Playback>。

## 安装 Wine 的一些注意事项

(1) 需要启用 multilib 仓库。

(2) 安装的包及描述：

| 包 | 描述 |
| ---- | ---- |
| wine | Wine 主程序 |
| wine-mono | .NET 程序支持 |
| winetricks | 一个魔改程序 |
| lib32-alsa-lib | ALSA 声音驱动包 |
| lib32-alsa-plugins | 上同 |

(3) 配置文件夹路径：`~/.wine/`

(4) 配置缩放

运行 winecfg，将 DPI 改为 (96 * 放大倍数)。

## 国内环境获取 oh-my-zsh

请访问 <https://help.mirrors.cernet.edu.cn/ohmyzsh.git/>。

> 什么是 **Oh My Zsh**？
>
> “Oh My Zsh” is a delightful, open source, community-driven framework for managing your Zsh configuration. It comes bundled with thousands of helpful functions, helpers, plugins, themes, and a few things that make you shout...

# Z Shell 的一些主题或者插件的 GitHub 地址

* Powerlevel10k 主题: <https://github.com/romkatv/powerlevel10k.git>

```bash
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k --depth=1
```

* Zsh 自动建议: <https://github.com/zsh-users/zsh-autosuggestions.git>

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions --depth=1
```

* Zsh 语法高亮: <https://github.com/zsh-users/zsh-syntax-highlighting.git>

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting --depth=1
```
