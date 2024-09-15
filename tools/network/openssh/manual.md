ssh(1) — 基本的 rlogin/rsh 类客户端程序

sshd(8) — 允许你登录的守护进程

ssh_config(5) — 客户端配置文件

sshd_config(5) — 守护进程配置文件

ssh-agent(1) — 可以存储私钥的身份验证代理

ssh-add(1) — 向上述代理添加密钥的工具

sftp(1) — 通过 SSH1 和 SSH2 协议运行的类似 FTP 的程序

scp(1) — 类似 rcp 的文件复制程序

ssh-keygen(1) — 密钥生成工具

sftp-server(8) — SFTP 服务器子系统（由 sshd 自动启动）

ssh-keyscan(1) — 用于从多个主机收集公钥的实用程序

ssh-keysign(8) — 基于主机的身份验证的辅助程序

OpenSSH 中实现的 SSH2 协议由 IETF secsh工作组标准化，并在多个 RFC 和草案中指定。SSH2 的总体结构在体系结构RFC 中描述 。它由三层组件组成：

    传输层（transport layer）提供算法协商和密钥交换。密钥交换包括服务器身份验证，并产生加密安全连接：它提供完整性、机密性和可选压缩。

    用户身份验证层（user authentication layer）使用已建立的连接并依赖于传输层提供的服务。它提供了几种用户身份验证机制。这些机制包括传统的密码身份验证以及公钥或基于主机的身份验证机制。

    连接层（connection layer）在经过身份验证的连接上多路复用许多不同的并发通道，并允许通过隧道传输登录会话和 TCP 转发。它为这些通道提供流量控制服务。此外，还可以协商各种特定于通道的选项。

