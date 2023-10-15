# 使用 Arch 时可能会出现的各种问题

## 将硬件时间设置为 localtime

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
