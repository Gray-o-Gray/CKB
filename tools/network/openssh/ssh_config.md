# ssh_config 配置文件
OpenSSH客户端配置文件

## ssh按下列顺序从以下来源获取配置数据
1. 命令行选项
2. 用户的配置文件 (~/.ssh/config)
3. 系统范围的配置文件 (/etc/ssh/ssh_config)
   
除非另有说明，对于每个参数，将使用第一个获得的值。

配置文件包含由主机规范分隔的部分，并且该部分仅适用于与规范中给出的模式之一匹配的主机。匹配的主机名通常是命令行中给出的主机名（有关例外情况，请参阅 CanonicalizeHostname 选项）。
```
Host *
    Port 22
```
由于使用了每个参数的第一个获得的值，因此应在文件开头附近给出更多特定于主机的声明，并在末尾给出一般默认值。

该文件包含关键字-参数对，每行一个。以“#”开头的行和空行将被解释为注释。参数可以选择用双引号 (") 括起来，以表示包含空格的参数。配置选项可以用 空格 或 可选空格和一个“=”分隔；后一种格式有助于避免在使用 ssh、scp 和 sftp -o 选项指定配置选项需要引用空格时出现匹配问题。

**实际应用场景**
1. 避免文件编辑：
   
    有时你可能不希望或不能编辑 SSH 的配置文件，那么可以在 SSH 命令中直接指定：

    ```
    ssh -o "PasswordAuthentication=no" username@server_ip
    ```

2. 解决空格问题：
    
    在处理含空格的选项值时，等号变得非常有用。例如:

    ```
    ssh -o "IdentityFile=~/.ssh/my id_rsa" username@server_ip  # 如果路径含空格，用等号可以避免混淆
    ```

    通过使用等号，避免了将路径（或其他值）用引号括起来可能带来的复杂性。

总结，在 OpenSSH 的配置选项中，你可以使用空格或等号来分隔配置项。这种灵活性不仅适用于配置文件，同样也适用于通过命令行参数 -o 来传递配置选项，提升了配置的灵活性和易用性。

## 可能的关键字及其含义
（请注意，关键字不区分大小写，而参数区分大小写）：

### Host

将以下声明（直到下一个 Host或Match关键字）限制为仅针对与关键字后给出的模式之一匹配的主机。如果提供了多个模式，则应以空格分隔它们。单个“ *”作为模式可用于为所有主机提供全局默认值。主机通常是命令行上给出的主机名CanonicalizeHostname参数（有关例外情况，请参阅关键字）。

通过在模式条目前加上感叹号 ('!') 可以将其否定。如果否定条目匹配，则Host忽略该条目，无论该行上是否有其他模式匹配。因此，否定匹配可用于为通配符匹配提供例外。

**示例解释**
1. 全局默认设置：
   
    你可以使用 Host * 定义全局默认设置，这些设置适用于所有没有明确指定的主机。
    ```
    Host *
        ForwardAgent no
        ForwardX11 no
        ServerAliveInterval 60
    ```
2. 特定域：
    
    为特定域的所有主机配置一个特定的用户和认证文件：
    ```
    Host *.example.com
        User myuser
        IdentityFile ~/.ssh/id_rsa_example
    ```
3. 否定模式：
   
    你可以使用否定模式排除特定主机或模式：
    ```
    Host !unwanted.example.com *.example.com
        User otheruser
        IdentityFile ~/.ssh/id_rsa_other
    ```

    在这个例子中，*.example.com 范围内的所有主机都将使用 otheruser 和指定的 IdentityFile，除了 unwanted.example.com。
4. 匹配多个模式：
    
    你也可以在同一行上匹配多个模式，用空格分隔：
    ```
    Host host1.example.com host2.example.com
        User specializeduser
        IdentityFile ~/.ssh/special_id_rsa
    ```

有关模式的更多信息，请参阅[PATTERNS](#模式patterns)。

### Match

限制以下声明（直到下一个 Host 或 Match 关键字）仅在满足 Match 关键字后的条件时使用。使用一个或多个条件或始终匹配的单个标记 all 指定匹配条件。可用的条件关键字为：canonical、final、exec、localnetwork、host、originalhost、tagged、user 和 localuser。all 条件必须单独出现或紧跟在 canonical 或 final 之后。其他条件可以任意组合。除 all、canonical 和 final 之外的所有条件都需要参数。可以通过在前面添加感叹号 ('!') 来否定条件。

canonical 关键字仅在主机名规范化后重新解析配置文件时匹配（请参阅 CanonicalizeHostname 选项）。这可能有助于指定仅适用于规范主机名的条件。

final 关键字要求重新解析配置（无论是否启用了 CanonicalizeHostname），并且仅在此最终传递过程中匹配。如果启用了 CanonicalizeHostname，则 canonical 和 final 会在同一传递过程中匹配。

exec 关键字在用户的 shell 下执行指定的命令。如果命令返回零退出状态，则条件视为真。包含空格字符的命令必须用引号引起来。exec 的参数接受 [TOKENS](#token) 部分中描述的标记。

localnetwork 关键字将活动本地网络接口的地址与提供的 CIDR 格式的网络列表进行匹配。这可能便于在网络间漫游的设备上改变有效配置。请注意，网络地址在许多情况下并不是一个值得信赖的标准（例如，当使用 DHCP 自动配置网络时），因此如果使用它来控制安全敏感配置，则应谨慎行事。

其他关键字的条件必须是单个条目或逗号分隔的列表，并且可以使用 [PATTERNS](#模式patterns) 部分中描述的通配符和否定运算符。
- 在 Hostname 或 CanonicalizeHostname 选项进行任何替换后，host 关键字的条件与目标主机名匹配。
- originalhost 关键字与在命令行上指定的主机名匹配。
- tagged 关键字与由之前的 Tag 指令或在 ssh 命令行上使用 -P 标志指定的标签名称匹配。user 关键字与远程主机上的目标用户名匹配。
- localuser 关键字与运行 ssh 的本地用户的名称匹配（此关键字可能在系统范围的 ssh_config 文件中有用）。

**单一条件关键字示例解释**
1. all: 
    
    始终匹配，必须单独使用或与 canonical、final 组合使用。
    ```
    Match all
        LogLevel VERBOSE
    ```
2. canonical: 

    当配置文件在主机名规范化之后重新解析时匹配。这个关键字适用于只对规范化的主机名工作的条件。
    ```
    Match canonical
        User canonicaluser
    ```
3. final: 
   
    请求重新解析配置，只有在最终解析过程中匹配。如果 CanonicalizeHostname 被启用，那么 canonical 和 final 在同一过程中匹配。
    ```
    Match final
        User finaluser
    ```
4. exec command:
   
    使用用户的 shell 执行指定的命令。如果命令返回零退出状态，则条件为真。包含空白字符的命令需要用引号括起来。命令的参数可以使用 [TOKENS](#token) 部分描述的令牌。
    ```
    Match exec "test -f /etc/ssh/allow_ssh"
        AllowTcpForwarding yes
    ```

5. localnetwork network/masklen: 
   
    将活动本地网络接口的地址与提供的CIDR格式的网络列表进行匹配。对于在不同网络间漫游的设备来说，这可能很方便。然而，在许多情况下（例如，通过 DHCP 自动配置网络时），网络地址不是一个可信的准则，因此在使用它控制安全相关配置时应谨慎。

    ```
    Match localnetwork 192.168.1.0/24,10.0.0.0/8
        User localnetworkuser
    ```

6. host hostname: 
    
    将目标主机名与指定的模式进行匹配。该模式可以使用 PATTERNS 部分描述的通配符和否定操作符。

    ```
    Match host *.example.com
        User exampleuser
        IdentityFile ~/.ssh/id_rsa_example
    ```

7. originalhost hostname: 
    
    将指定的主机名与命令行上的主机名进行匹配。该模式可以使用 PATTERNS 部分描述的通配符和否定操作符。

    ```
    Match originalhost server1.example.com
        User originalhostuser
    ```

8. tagged tag:
    
    将给定的标签名与先前的 Tag 指令或在 ssh 命令行上使用 -P 标志指定的标签进行匹配。

    ```
    Match tagged sometag
        User taggeduser
    ```

9. user username: 
    
    将目标用户名与指定的用户名进行匹配。该模式可以使用 PATTERNS 部分描述的通配符和否定操作符。

    ```
    Match user admin
        PermitTunnel yes
    ```

10. localuser username: 
    
    将本地运行 ssh 的用户与指定的用户名进行匹配。该模式可以使用 PATTERNS 部分描述的通配符和否定操作符。这个关键字在系统范围的 ssh_config 文件中尤为有用。
    ```
    Match localuser jeff
        IdentityFile ~/.ssh/id_rsa_jeff
    ```
11. 否定条件

    条件可以通过前缀 ! 来否定：
    ```
    Match user !bob
        AllowTcpForwarding yes
    ```
    
    这表示当用户不是 bob 时允许 TCP 转发。

**综合示例**

以下是一个综合示例，展示如何组合多个条件使用 Match 关键字：

```
# 全局默认配置
Host *
    ForwardAgent no
    ForwardX11 no
    ServerAliveInterval 60

# 域 example.com 的主机
Match host *.example.com
    User exampleuser
    IdentityFile ~/.ssh/id_rsa_example

# 排除 unwanted.example.com
Match host !unwanted.example.com *.example.com
    User otheruser
    IdentityFile ~/.ssh/id_rsa_other

# 用户为 admin 时
Match user admin
    PermitTunnel yes

# 满足特定执行条件
Match exec "uname -a | grep Linux"
    User linuxuser
    IdentityFile ~/.ssh/id_rsa_linux

# 用户为 alice 且主机为 server1.example.com 时
Match user alice host server1.example.com
    User alice
    IdentityFile ~/.ssh/id_rsa_alice
    ForwardAgent yes

# 本地网络匹配
Match localnetwork 192.168.1.0/24,10.0.0.0/8
    User localnetworkuser

# 本地用户为 jeff 时
Match localuser jeff
    IdentityFile ~/.ssh/id_rsa_jeff
```

### AddKeysToAgent

Specifies whether keys should be automatically added to a running ssh-agent. If this option is set to yes and a key is loaded from a file, the key and its passphrase are added to the agent with the default lifetime, as if by ssh-add. If this option is set to ask, ssh will require confirmation using the SSH_ASKPASS program before adding a key (see ssh-add for details). If this option is set to confirm, each use of the key must be confirmed, as if the -c option was specified to ssh-add. If this option is set to no, no keys are added to the agent. Alternately, this option may be specified as a time interval using the format described in the TIME FORMATS section of sshd_config to specify the key's lifetime in ssh-agent, after which it will automatically be removed. The argument must be no (the default), yes, confirm (optionally followed by a time interval), ask or a time interval.

指定是否应自动将密钥添加到正在运行的 ssh-agent。

如果将此选项设置为 yes，并且密钥是从文件加载的，则密钥及其密码将以默认生存期添加到代理，就像通过 ssh-add 一样。

如果将此选项设置为 ask，ssh(1) 将要求在添加密钥之前使用 SSH_ASKPASS 程序进行确认（有关详细信息，请参阅 ssh-add）。

如果将此选项设置为 confirmed，则每次使用密钥时都必须确认，就像为 ssh-add 指定了 -c 选项一样。

如果将此选项设置为 no，则不会将任何密钥添加到代理。或者，可以使用 sshd_config 的 TIME FORMATS 部分中描述的格式将此选项指定为时间间隔，以指定密钥在 ssh-agent 中的生存期，在此之后将自动删除密钥。

**参数必须为否（默认）、是、确认（可选择后跟时间间隔）、询问或时间间隔之一。**

### AddressFamily

Specifies which address family to use when connecting. Valid arguments are any (the default), inet (use IPv4 only), or inet6 (use IPv6 only).

指定连接时要使用的地址系列。有效参数为 any（默认）、inet（仅使用 IPv4）或 inet6（仅使用 IPv6）。

## 模式（PATTERNS）

## TOKEN