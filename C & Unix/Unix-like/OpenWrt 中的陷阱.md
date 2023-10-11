# OpenWrt 中的陷阱

## SSH 访问

OpenWrt 默认使用 Dropbear 提供 SSH 访问和 SCP 服务，而不是 OpenSSH，所以 `authorized_keys` 文件应被放置于 `/etc/dropbear` 目录。
