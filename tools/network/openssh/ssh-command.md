# ssh基础命令

## 连接远程主机
```
ssh user@hostname 

ssh user@hostname -p port
```


## 概要
```
ssh	[-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface] [-b bind_address] [-c cipher_spec] [-D [bind_address:]port] [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11] [-i identity_file] [-J destination] [-L address] [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-P tag] [-p port] [-R address] [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]] destination [command [argument ...]]

ssh	[-Q query_option]

```
## 描述
ssh（SSH 客户端）是一种用于登录远程计算机并在远程计算机上执行命令的程序。它旨在通过不安全的网络在两个不受信任的主机之间提供安全的加密通信。X11 连接、任意 TCP 端口和 UNIX域套接字也可以通过安全通道转发。

ssh连接并登录到指定的 目标，该目标可以指定为 [user@]hostname 或 ssh:// [user@]hostname[:port] 形式的 URI。用户必须使用以下几种方法之一向远程计算机证明其身份（见下文）。

如果指定了命令，它将在远程主机上执行，而不是在登录 shell 上执行。完整的命令行可以指定为command，也可以包含其他参数。如果提供了参数，则参数将附加到命令中，以空格分隔，然后再发送到服务器执行。





