# OpenSSH 简介

OpenSSH 是使用 SSH 协议进行远程登录的首要连接工具。它加密所有流量以消除窃听、连接劫持和其他攻击。此外，OpenSSH 还提供大量安全隧道功能、多种身份验证方法和复杂的配置选项。

OpenSSH 套件由以下工具组成：

远程操作使用ssh、 scp和 sftp 完成。

使用ssh-add、 ssh-keysign、 ssh-keyscan和 ssh-keygen 进行密钥管理。

服务端由sshd、 sftp-server和 ssh-agent组成 。

## 验证主机密钥
首次连接到服务器时，会向用户显示服务器公钥的指纹（除非 StrictHostKeyChecking已禁用该选项）。可以使用 ssh-keygen(1)确定指纹：
```
ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key
```
如果指纹已知，则可以匹配，并接受或拒绝密钥。如果只有服务器的传统 (MD5) 指纹可用，则可以使用ssh-keygen -E选项降级指纹算法以进行匹配。


