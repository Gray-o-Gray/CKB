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

key : parameter

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

### BatchMode

If set to yes, user interaction such as password prompts and host key confirmation requests will be disabled. This option is useful in scripts and other batch jobs where no user is present to interact with ssh. The argument must be yes or no (the default).

如果设置为 yes，将禁用用户交互，例如密码提示和主机密钥确认请求。这个选项在没有用户进行交互的脚本和其他批处理作业中非常有用。参数必须为 yes 或 no（默认值）。

### BindAddress

Use the specified address on the local machine as the source address of the connection. Only useful on systems with more than one address.

使用本地机器上指定的地址作为连接的源地址。仅在具有多个地址的系统上有用。

### BindInterface

Use the address of the specified interface on the local machine as the source address of the connection.

使用本地机器上指定接口的地址作为连接的源地址。

### CanonicalDomains

When CanonicalizeHostname is enabled, this option specifies the list of domain suffixes in which to search for the specified destination host.

启用 CanonicalizeHostname 时，此选项指定要搜索指定目标主机的域后缀列表。

### CanonicalizeFallbackLocal

Specifies whether to fail with an error when hostname canonicalization fails. The default, yes, will attempt to look up the unqualified hostname using the system resolver's search rules. A value of no will cause ssh to fail instantly if CanonicalizeHostname is enabled and the target hostname cannot be found in any of the domains specified by CanonicalDomains.

指定当主机名规范化失败时是否返回错误。默认值为 yes，会尝试使用系统解析程序的搜索规则查找不合格的主机名。如果设置为 no，并且启用了 CanonicalizeHostname，当无法在任何 CanonicalDomains 指定的域中找到目标主机名时，ssh将立即失败。

#### 主机名规范化

主机名规范化是将不完整或简化的主机名转化为一个完整的、标准化的格式，以确保能够正确解析或连接。通过在指定的域后缀列表中搜索，主机名规范化试图找到并补全目标主机名，使其符合网络地址解析的要求。

**示例**

假设你要连接的主机名是 server，但它并不是一个完全限定的域名（FQDN）。如果启用了主机名规范化，并配置了域后缀列表如 example.com 和 example.org，系统会尝试依次解析 server.example.com 和 server.example.org，直到找到有效的主机名为止。这样就完成了主机名的规范化。

### CanonicalizeHostname

Controls whether explicit hostname canonicalization is performed. The default, no, is not to perform any name rewriting and let the system resolver handle all hostname lookups. If set to yes then, for connections that do not use a ProxyCommand or ProxyJump, ssh(1) will attempt to canonicalize the hostname specified on the command line using the CanonicalDomains suffixes and CanonicalizePermittedCNAMEs rules. If CanonicalizeHostname is set to always, then canonicalization is applied to proxied connections too.

控制是否执行显式主机名规范化。默认值为 no，不进行任何名称重写，让系统解析器处理所有主机名查找。如果设置为 yes，对于未使用 ProxyCommand 或 ProxyJump 的连接，ssh将使用 CanonicalDomains 后缀和 CanonicalizePermittedCNAMEs 规则尝试规范化命令行中指定的主机名。如果将 CanonicalizeHostname 设置为 always，则对代理连接也应用规范化。

If this option is enabled, then the configuration files are processed again using the new target name to pick up any new configuration in matching Host and Match stanzas. A value of none disables the use of a ProxyJump host.

如果启用此选项，配置文件将使用新的目标名称再次处理，以获取在匹配的 Host 和 Match 段中的任何新配置。值为 none 时，禁用使用 ProxyJump 主机。

### CanonicalizeMaxDots

Specifies the maximum number of dot characters in a hostname before canonicalization is disabled. The default, 1, allows a single dot (i.e. hostname.subdomain).

指定在禁用规范化之前主机名中点字符的最大数量。默认值为 1，允许单个点（例如，hostname.subdomain）。

### CanonicalizePermittedCNAMEs

Specifies rules to determine whether CNAMEs should be followed when canonicalizing hostnames. The rules consist of one or more arguments of source_domain_list:target_domain_list, where source_domain_list is a pattern-list of domains that may follow CNAMEs in canonicalization, and target_domain_list is a pattern-list of domains that they may resolve to.

指定规则以确定在规范化主机名时是否应遵循 CNAME。这些规则由一个或多个 source_domain_list:target_domain_list 参数组成，其中 source_domain_list 是可以在规范化中跟随 CNAME 的域的模式列表，target_domain_list 是它们可以解析到的域的模式列表。

For example, "*.a.example.com:*.b.example.com,*.c.example.com" will allow hostnames matching "*.a.example.com" to be canonicalized to names in the "*.b.example.com" or "*.c.example.com" domains.

例如，"*.a.example.com:*.b.example.com,*.c.example.com" 允许与 "*.a.example.com" 匹配的主机名规范化为 "*.b.example.com" 或 "*.c.example.com" 域中的名称。

A single argument of "none" causes no CNAMEs to be considered for canonicalization. This is the default behaviour.

单个参数 "none" 表示不考虑任何 CNAME 进行规范化。这是默认行为。

## CASignatureAlgorithms

Specifies which algorithms are allowed for signing of certificates by certificate authorities (CAs). The default is:

指定允许证书颁发机构（CAs）用来签署证书的算法。默认为：

- ssh-ed25519,ecdsa-sha2-nistp256,
- ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
- sk-ssh-ed25519@openssh.com,
- sk-ecdsa-sha2-nistp256@openssh.com,
- rsa-sha2-512,rsa-sha2-256

If the specified list begins with a ‘+’ character, then the specified algorithms will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified algorithms (including wildcards) will be removed from the default set instead of replacing them.

如果指定的列表以字符 ‘+’ 开头，则所指定的算法将附加到默认集合，而不是替换它们。如果列表以字符 ‘-’ 开头，则所指定的算法（包括通配符）将从默认集合中删除，而不是替换它们。

ssh(1) will not accept host certificates signed using algorithms other than those specified.

ssh 不会接受使用未指定算法签署的主机证书。

### CertificateFile

Specifies a file from which the user's certificate is read. A corresponding private key must be provided separately in order to use this certificate either from an IdentityFile directive or -i flag to ssh(1), via ssh-agent(1), or via a PKCS11Provider or SecurityKeyProvider.

指定一个文件以读取用户的证书。为了使用此证书，必须单独提供相应的私钥，可以通过 IdentityFile 指令或 -i 标志、ssh-agent、PKCS11Provider 或 SecurityKeyProvider 来提供。

Arguments to CertificateFile may use the tilde syntax to refer to a user's home directory, the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section.

CertificateFile 的参数可以使用波浪号语法来引用用户的主目录，还可以使用在 TOKENS 部分描述的令牌和在 ENVIRONMENT VARIABLES 部分描述的环境变量。

It is possible to have multiple certificate files specified in configuration files; these certificates will be tried in sequence. Multiple CertificateFile directives will add to the list of certificates used for authentication.

可以在配置文件中指定多个证书文件，这些证书将按顺序尝试。多个 CertificateFile 指令会将证书添加到用于身份验证的列表中。

### ChannelTimeout

Specifies whether and how quickly ssh(1) should close inactive channels. Timeouts are specified as one or more “type=interval” pairs separated by whitespace, where the “type” must be the special keyword “global” or a channel type name from the list below, optionally containing wildcard characters.

指定 ssh 是否以及多快关闭不活跃的通道。超时以一个或多个 “type=interval” 对的形式指定，以空格分隔，其中 “type” 必须是特殊关键字 “global” 或下列通道类型名称，可以包含通配符。

The timeout value “interval” is specified in seconds or may use any of the units documented in the TIME FORMATS section. For example, “session=5m” would cause interactive sessions to terminate after five minutes of inactivity. Specifying a zero value disables the inactivity timeout.

超时值 “interval” 可以用秒表示，也可以使用在 TIME FORMATS 部分记录的任何单位。例如，"session=5m" 将导致交互会话在五分钟不活动后终止。指定零值将禁用不活动超时。

The special timeout “global” applies to all active channels, taken together. Traffic on any active channel will reset the timeout, but when the timeout expires then all open channels will be closed. Note that this global timeout is not matched by wildcards and must be specified explicitly.

特殊超时 “global” 适用于所有活动通道。任何活动通道上的流量都会重置超时，但一旦超时到期，所有打开的通道都将关闭。注意，通配符不会匹配此全局超时，必须明确指定。

The available channel type names include:
- agent-connection
  
  Open connections to ssh-agent(1).
- direct-tcpip, direct-streamlocal@openssh.com
  
  Open TCP or Unix socket (respectively) connections that have been established from a ssh(1) local forwarding, i.e. LocalForward or DynamicForward.
- forwarded-tcpip, forwarded-streamlocal@openssh.com
  
  Open TCP or Unix socket (respectively) connections that have been established to a sshd(8) listening on behalf of a ssh(1) remote forwarding, i.e. RemoteForward.
- session
  
  The interactive main session, including shell session, command execution, scp(1), sftp(1), etc.
- tun-connection
  
  Open TunnelForward connections.
- x11-connection
  
  Open X11 forwarding sessions.

Note that in all the above cases, terminating an inactive session does not guarantee to remove all resources associated with the session, e.g. shell processes or X11 clients relating to the session may continue to execute.

可用的通道类型名称包括：
- agent-connection：打开与 ssh-agent(1) 的连接。
- direct-tcpip, direct-streamlocal@openssh.com：打开从 ssh(1) 本地转发建立的 TCP 或 Unix 套接字连接，如 LocalForward 或 DynamicForward。
- forwarded-tcpip, forwarded-streamlocal@openssh.com：打开为 ssh(1) 远程转发而代表 sshd(8) 侦听的 TCP 或 Unix 套接字连接，如 RemoteForward。
- session：交互式主会话，包括 shell 会话、命令执行、scp(1)、sftp(1) 等。
- tun-connection：打开 TunnelForward 连接。
- x11-connection：打开 X11 转发会话。

在上述所有情况下，终止不活跃会话并不能保证移除与会话关联的所有资源，例如，与会话相关的 shell 进程或 X11 客户端可能会继续执行。

Moreover, terminating an inactive channel or session does not necessarily close the SSH connection, nor does it prevent a client from requesting another channel of the same type. In particular, expiring an inactive forwarding session does not prevent another identical forwarding from being subsequently created.

此外，终止不活跃的通道或会话并不一定会关闭 SSH 连接，也不会阻止客户端请求同类型的其他通道。特别是，终止不活跃的转发会话不会阻止随后创建另一个相同的转发。

The default is not to expire channels of any type for inactivity.

默认情况下，不会因不活动而终止任何类型的通道。

### CheckHostIP

If set to yes, ssh(1) will additionally check the host IP address in the known_hosts file. This allows it to detect if a host key changed due to DNS spoofing and will add addresses of destination hosts to ~/.ssh/known_hosts in the process, regardless of the setting of StrictHostKeyChecking. If the option is set to no (the default), the check will not be executed.

如果设置为 yes，ssh 将额外检查 known_hosts 文件中的主机 IP 地址。这可以检测主机密钥是否因 DNS 欺骗而更改，并将在此过程中将目标主机的地址添加到 ~/.ssh/known_hosts 中，而不考虑 StrictHostKeyChecking 的设置。如果选项设置为 no（默认），则不会执行此检查。

### Ciphers

Specifies the ciphers allowed and their order of preference. Multiple ciphers must be comma-separated. If the specified list begins with a ‘+’ character, then the specified ciphers will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified ciphers (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified ciphers will be placed at the head of the default set.

指定允许的加密算法及其优先顺序。多个算法需用逗号分隔。如果指定列表以 ‘+’ 开头，说明这些算法将添加到默认集而不是替换它们。如果以 ‘-’ 开头，说明这些算法（包括通配符）将从默认集中移除而不是替换它们。如果以 ‘^’ 开头，说明这些算法将置于默认集的开头。

The supported ciphers are:
- 3des-cbc
- aes128-cbc
- aes192-cbc
- aes256-cbc
- aes128-ctr
- aes192-ctr
- aes256-ctr
- aes128-gcm@openssh.com
- aes256-gcm@openssh.com
- chacha20-poly1305@openssh.com

The default is:
- chacha20-poly1305@openssh.com,
- aes128-ctr,aes192-ctr,aes256-ctr,
- aes128-gcm@openssh.com,aes256-gcm@openssh.com

The list of available ciphers may also be obtained using "ssh -Q cipher".

可以使用命令 ssh -Q cipher 来获取可用加密算法的列表。

### ClearAllForwardings

Specifies that all local, remote, and dynamic port forwardings specified in the configuration files or on the command line be cleared. This option is primarily useful when used from the ssh(1) command line to clear port forwardings set in configuration files, and is automatically set by scp(1) and sftp(1). The argument must be yes or no (the default).

指定清除配置文件或命令行中所有本地、远程和动态端口转发。当从 ssh(1) 命令行使用以清除配置文件中设置的端口转发时，这个选项非常有用，并且由 scp(1) 和 sftp(1) 自动设置。参数必须是 yes 或 no（默认）。

### Compression

Specifies whether to use compression. The argument must be yes or no (the default).

指定是否使用压缩。参数必须是 yes 或 no（默认）。

### ConnectionAttempts

Specifies the number of tries (one per second) to make before exiting. The argument must be an integer. This may be useful in scripts if the connection sometimes fails. The default is 1.

指定在退出前尝试的次数（每秒一次）。参数必须是整数。这在连接有时失败的情况下，对脚本可能很有用。默认值是 1。

### ConnectTimeout

Specifies the timeout (in seconds) used when connecting to the SSH server, instead of using the default system TCP timeout. This timeout is applied both to establishing the connection and to performing the initial SSH protocol handshake and key exchange.

指定连接到 SSH 服务器时使用的超时时间（秒），而不是使用系统的默认 TCP 超时。此超时适用于建立连接以及执行初始 SSH 协议握手和密钥交换。

### ControlMaster

Enables the sharing of multiple sessions over a single network connection. When set to yes, ssh(1) will listen for connections on a control socket specified using the ControlPath argument. Additional sessions can connect to this socket using the same ControlPath with ControlMaster set to no (the default). These sessions will try to reuse the master instance's network connection rather than initiating new ones, but will fall back to connecting normally if the control socket does not exist, or is not listening.

启用在单个网络连接上共享多个会话。设置为 yes 时，ssh 会根据 ControlPath 参数指定的控制套接字监听连接。其他会话可以使用相同的 ControlPath，并将 ControlMaster 设置为 no（默认值）来连接这个套接字。这些会话会尝试重用主实例的网络连接，而不是发起新连接；如果控制套接字不存在或未监听，则会回退到正常连接。

Setting this to ask will cause ssh(1) to listen for control connections, but require confirmation using ssh-askpass(1). If the ControlPath cannot be opened, ssh(1) will continue without connecting to a master instance.

将其设置为 ask 会让 ssh 监听控制连接，但需要使用 ssh-askpass 进行确认。如果无法打开 ControlPath，ssh 将在不连接到主实例的情况下继续。

**主实例**
```
主实例是在启用 ControlMaster 时，首次建立的 SSH 连接实例。

主实例的作用：
共享连接：建立后，其他会话可以复用主实例的网络连接，大大减少延迟和资源。
控制套接字：主实例创建控制套接字，允许其他会话连接。

比如，你首次连接使用 ControlMaster 配置成为主实例，之后的连接可以复用这个实例的连接状态，从而避免多次握手和认证过程。
```

X11 and ssh-agent(1) forwarding is supported over these multiplexed connections, however the display and agent forwarded will be the one belonging to the master connection i.e. it is not possible to forward multiple displays or agents.

在这些多路复用连接上支持 X11 和 ssh-agent 转发，但显示和代理将使用主连接的，这意味着无法转发多个显示或代理。

**示例**
```

假设你通过 SSH 连接到了 example.com，并启用了控制主实例：

ssh -o ControlMaster=yes -o ControlPath=~/.ssh/control-%r@%h:%p example.com
​
在这个连接中，你启用了 X11 转发和 ssh-agent 转发。

如果你在同一台机器上再次连接到 example.com，并且该连接使用了同一个控制套接字：

ssh -o ControlPath=~/.ssh/control-%r@%h:%p example.com
​
在这第二个会话中，由于连接复用了主实例，因此它将使用主实例的 X11 显示和 ssh-agent 转发。这意味着如果需要不同的 X11 显示或代理，它们将无法在次要连接中单独转发。
```

Two additional options allow for opportunistic multiplexing: try to use a master connection but fall back to creating a new one if one does not already exist. These options are: auto and autoask. The latter requires confirmation like the ask option.

有两个附加选项可以实现机会性多路复用：尝试使用主连接，但如果不存在则创建新连接。这些选项是：auto 和 autoask。后者需要像 ask 选项一样确认。

### ControlPath

Specify the path to the control socket used for connection sharing as described in the ControlMaster section above or the string none to disable connection sharing. Arguments to ControlPath may use the tilde syntax to refer to a user's home directory, the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section. It is recommended that any ControlPath used for opportunistic connection sharing include at least %h, %p, and %r (or alternatively %C) and be placed in a directory that is not writable by other users. This ensures that shared connections are uniquely identified.

请指定用于连接共享的控制套接字路径，如上面的 ControlMaster 部分所述，或使用字符串 "none" 禁用连接共享。ControlPath 的参数可以使用波浪号语法来引用用户的主目录，还可以使用 TOKENS 部分中描述的标记和 ENVIRONMENT VARIABLES 部分中描述的环境变量。建议任何用于机会连接共享的 ControlPath 至少包含 %h、%p 和 %r（或替代地 %C），并且放置在其他用户无写入权限的目录中。这可确保共享连接具有唯一标识。

### ControlPersist

When used in conjunction with ControlMaster, specifies that the master connection should remain open in the background (waiting for future client connections) after the initial client connection has been closed. If set to no (the default), then the master connection will not be placed into the background, and will close as soon as the initial client connection is closed. If set to yes or 0, then the master connection will remain in the background indefinitely (until killed or closed via a mechanism such as the "ssh -O exit"). If set to a time in seconds, or a time in any of the formats documented in sshd_config(5), then the backgrounded master connection will automatically terminate after it has remained idle (with no client connections) for the specified time.

与ControlMaster一起使用时，指定主连接在初始客户端连接关闭后应在后台保持开启，以等待未来的客户端连接。如果设置为no（默认），则主连接不会进入后台，并将在初始客户端连接关闭后立即关闭。如果设置为yes或0，则主连接将无限期保持在后台（直到被终止或通过如“ssh -O exit”机制关闭）。如果设置为以秒为单位的时间，或sshd_config中记录的格式之一，则后台主连接在空闲达到指定时间后会自动终止（无客户端连接）。

### DynamicForward

Specifies that a TCP port on the local machine be forwarded over the secure channel, and the application protocol is then used to determine where to connect to from the remote machine.

指定本地机器的TCP端口通过安全通道转发，然后使用应用协议决定从远程机器连接的位置。

The argument must be [bind_address:]port. IPv6 addresses can be specified by enclosing addresses in square brackets. By default, the local port is bound in accordance with the GatewayPorts setting. However, an explicit bind_address may be used to bind the connection to a specific address. The bind_address of localhost indicates that the listening port be bound for local use only, while an empty address or ‘*’ indicates that the port should be available from all interfaces.

参数必须是[bind_address:]port。IPv6地址可以通过方括号括起来指定。默认情况下，本地端口根据GatewayPorts设置绑定。但是，可以使用明确的bind_address将连接绑定到特定地址。localhost的bind_address表示监听端口仅供本地使用，而空地址或‘*’表示端口应可从所有接口访问。

Currently the SOCKS4 and SOCKS5 protocols are supported, and ssh(1) will act as a SOCKS server. Multiple forwardings may be specified, and additional forwardings can be given on the command line. Only the superuser can forward privileged ports.

当前支持SOCKS4和SOCKS5协议，ssh(1)会作为一个SOCKS服务器。可以指定多个转发，并可以在命令行中提供其他转发。只有超级用户可以转发特权端口。

### EnableEscapeCommandline

Enables the command line option in the EscapeChar menu for interactive sessions (default ‘~C’). By default, the command line is disabled.

在EscapeChar菜单中启用命令行选项用于交互会话（默认是‘~C’）。默认情况下，命令行是禁用的。

**示例：SSH会话中的EscapeChar命令行**

假设你正在使用SSH连接远程服务器，并且需要在会话中启用命令行选项。

1. **默认设置**  
   默认情况下，命令行选项是禁用的。

2. **启用选项**  
   通过设置EscapeChar为`~C`，可以启用命令行选项。

3. **使用示例**  
   - 连接到服务器：  
     ```bash
     ssh user@server.com
     ```
   - 在会话中按下`~C`：  
     这将调用命令行，让你可以执行一些特殊命令，如端口转发或重新连接。

4. **退出命令行**  
   执行完需要的操作后，返回到普通会话中继续使用SSH。

通过这种方式，你可以在不中断SSH连接的情况下完成其他任务。

### EnableSSHKeysign

Setting this option to yes in the global client configuration file /etc/ssh/ssh_config enables the use of the helper program ssh-keysign(8) during HostbasedAuthentication. The argument must be yes or no (the default). This option should be placed in the non-hostspecific section. See ssh-keysign(8) for more information.

在全局客户端配置文件 /etc/ssh/ssh_config 中，将此选项设置为 yes，可以在 HostbasedAuthentication 中使用辅助程序 ssh-keysign(8)。参数必须为 yes 或 no（默认）。此选项应放在非主机特定部分。详细信息请参阅 ssh-keysign(8)。

### EscapeChar

Sets the escape character (default: ‘~’). The escape character can also be set on the command line. The argument should be a single character, ‘^’ followed by a letter, or none to disable the escape character entirely (making the connection transparent for binary data).

设置转义字符（默认值：‘~’）。转义字符也可以在命令行中设置。参数应为单个字符，可以是‘^’后跟一个字母，或者不设置以完全禁用转义字符（使连接对二进制数据透明）。

### ExitOnForwardFailure

Specifies whether ssh(1) should terminate the connection if it cannot set up all requested dynamic, tunnel, local, and remote port forwardings, (e.g. if either end is unable to bind and listen on a specified port). Note that ExitOnForwardFailure does not apply to connections made over port forwardings and will not, for example, cause ssh(1) to exit if TCP connections to the ultimate forwarding destination fail. The argument must be yes or no (the default).

指定当无法设置所有请求的动态、隧道、本地和远程端口转发时，ssh(1)是否应终止连接（例如，如果任一端无法在指定端口绑定和监听）。请注意，ExitOnForwardFailure不适用于通过端口转发建立的连接，不会因为到最终转发目标的TCP连接失败而导致ssh(1)退出。参数必须是yes或no（默认）。

### FingerprintHash

Specifies the hash algorithm used when displaying key fingerprints. Valid options are: md5 and sha256 (the default).

指定用于显示密钥指纹的哈希算法。有效选项有：md5 和 sha256（默认）。

### ForkAfterAuthentication

Requests ssh to go to background just before command execution. This is useful if ssh is going to ask for passwords or passphrases, but the user wants it in the background. This implies the StdinNull configuration option being set to “yes”. The recommended way to start X11 programs at a remote site is with something like ssh -f host xterm, which is the same as ssh host xterm if the ForkAfterAuthentication configuration option is set to “yes”.

请求在命令执行前让SSH进入后台。这在SSH需要输入密码或口令但用户希望其在后台运行时很有用。这意味着StdinNull配置选项设置为“yes”。推荐的方式是在远程启动X11程序时使用类似ssh -f host xterm的命令，如果ForkAfterAuthentication配置选项设为“yes”，这就等同于ssh host xterm。

If the ExitOnForwardFailure configuration option is set to “yes”, then a client started with the ForkAfterAuthentication configuration option being set to “yes” will wait for all remote port forwards to be successfully established before placing itself in the background. The argument to this keyword must be yes (same as the -f option) or no (the default).

如果将 ExitOnForwardFailure 配置选项设为“yes”，那么在 ForkAfterAuthentication 配置选项设为“yes”时启动的客户端将在所有远程端口转发成功建立后才会进入后台。此关键字的参数必须是“yes”（与 -f 选项相同）或“no”（默认值）。

### ForwardAgent

Specifies whether the connection to the authentication agent (if any) will be forwarded to the remote machine. The argument may be yes, no (the default), an explicit path to an agent socket or the name of an environment variable (beginning with ‘$’) in which to find the path.

指定是否将连接到身份验证代理（如果有的话）转发到远程机器。参数可以是“yes”、“no”（默认值）、代理套接字的明确路径或以“$”开头的环境变量名以找到路径。

Agent forwarding should be enabled with caution. Users with the ability to bypass file permissions on the remote host (for the agent's Unix-domain socket) can access the local agent through the forwarded connection. An attacker cannot obtain key material from the agent, however they can perform operations on the keys that enable them to authenticate using the identities loaded into the agent.

应谨慎启用代理转发。可以绕过远程主机上文件权限的用户（代理的Unix域套接字）能够通过转发连接访问本地代理。攻击者无法从代理中获取密钥材料，但可以对密钥执行操作，使他们能够使用加载到代理中的身份进行身份验证。

### ForwardX11

Specifies whether X11 connections will be automatically redirected over the secure channel and DISPLAY set. The argument must be yes or no (the default).

指定是否将 X11 连接自动重定向到安全通道，并设置 DISPLAY。参数必须是 "yes" 或 "no"（默认值）。

X11 forwarding should be enabled with caution. Users with the ability to bypass file permissions on the remote host (for the user's X11 authorization database) can access the local X11 display through the forwarded connection. An attacker may then be able to perform activities such as keystroke monitoring if the ForwardX11Trusted option is also enabled.

启用X11转发时应谨慎。能够绕过远程主机文件权限的用户（针对用户的X11授权数据库）可以通过转发连接访问本地X11显示。如果同时启用了ForwardX11Trusted选项，攻击者可能能够进行如击键监控等活动。

### ForwardX11Timeout

Specify a timeout for untrusted X11 forwarding using the format described in the TIME FORMATS section of sshd_config(5). X11 connections received by ssh(1) after this time will be refused. Setting ForwardX11Timeout to zero will disable the timeout and permit X11 forwarding for the life of the connection. The default is to disable untrusted X11 forwarding after twenty minutes has elapsed.

为不受信任的 X11 转发指定超时时间，使用 sshd_config 的时间格式描述。ssh(1) 接收到的 X11 连接在此时间之后将被拒绝。将 ForwardX11Timeout 设置为零将禁用超时，并允许在连接期间进行 X11 转发。默认情况下，二十分钟后禁用不受信任的 X11 转发。

### ForwardX11Trusted

If this option is set to yes, remote X11 clients will have full access to the original X11 display.

如果此选项设置为yes，远程X11客户端将对原始X11显示器有完全访问权限。

If this option is set to no (the default), remote X11 clients will be considered untrusted and prevented from stealing or tampering with data belonging to trusted X11 clients. Furthermore, the xauth(1) token used for the session will be set to expire after 20 minutes. Remote clients will be refused access after this time.

如果该选项设置为“no”（默认），远程 X11 客户端将被视为不受信任，无法窃取或篡改受信任 X11 客户端的数据。此外，session 使用的 xauth 令牌将在 20 分钟后过期。届时将拒绝远程客户端的访问。

See the X11 SECURITY extension specification for full details on the restrictions imposed on untrusted clients.

有关对不受信任客户端施加的限制的详细信息，请参阅 X11 SECURITY 扩展规范。

### GatewayPorts

Specifies whether remote hosts are allowed to connect to local forwarded ports. By default, ssh(1) binds local port forwardings to the loopback address. This prevents other remote hosts from connecting to forwarded ports. GatewayPorts can be used to specify that ssh should bind local port forwardings to the wildcard address, thus allowing remote hosts to connect to forwarded ports. The argument must be yes or no (the default).

指定是否允许远程主机连接到本地转发的端口。默认情况下，ssh将本地端口转发绑定到回环地址，这阻止其他远程主机连接到转发的端口。可以使用GatewayPorts指定ssh应将本地端口转发绑定到通配符地址，从而允许远程主机连接到转发的端口。参数必须是yes或no（默认值）。

### GlobalKnownHostsFile

Specifies one or more files to use for the global host key database, separated by whitespace. The default is /etc/ssh/ssh_known_hosts, /etc/ssh/ssh_known_hosts2.

指定一个或多个文件作全局主机密钥数据库，文件名以空格分隔。默认值为/etc/ssh/ssh_known_hosts和/etc/ssh/ssh_known_hosts2。

### GSSAPIAuthentication

Specifies whether user authentication based on GSSAPI is allowed. The default is no.

指定是否允许基于GSSAPI的用户认证。默认值为no。

#### GSSAPI
GSSAPI（通用安全服务应用程序编程接口）是一种用于在网络应用程序中提供安全认证和通信的标准接口。它通过抽象具体的安全机制，使应用程序能够使用多种认证方法，而无需了解底层的实现细节。GSSAPI常用于支持如Kerberos这样的身份验证协议，确保数据的完整性和保密性。

### GSSAPIDelegateCredentials

Forward (delegate) credentials to the server. The default is no.

将凭证转发（委托）到服务器。默认值为no。

### HashKnownHosts

Indicates that ssh(1) should hash host names and addresses when they are added to ~/.ssh/known_hosts. These hashed names may be used normally by ssh(1) and sshd(8), but they do not visually reveal identifying information if the file's contents are disclosed. The default is no. Note that existing names and addresses in known hosts files will not be converted automatically, but may be manually hashed using ssh-keygen(1).

表示在添加到 ~/.ssh/known_hosts 文件时，ssh 应对主机名和地址进行哈希处理。ssh 和 sshd 可以正常使用这些哈希后的名称，但如果文件内容泄露，它们不会透露识别信息。默认值为 no。注意，已存在于 known_hosts 文件中的名称和地址不会自动转换，但可以使用 ssh-keygen 手动哈希。

### HostbasedAcceptedAlgorithms

Specifies the signature algorithms that will be used for hostbased authentication as a comma-separated list of patterns. Alternately if the specified list begins with a ‘+’ character, then the specified signature algorithms will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified signature algorithms (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified signature algorithms will be placed at the head of the default set. The default for this option is:

指定用于基于主机身份验证的签名算法，以逗号分隔的模式列表表示。或者，如果指定列表以‘+’字符开始，那么指定的签名算法将被添加到默认集合中，而不是替换它们。如果指定列表以‘-’字符开始，那么指定的签名算法（包括通配符）将从默认集合中移除，而不是替换它们。如果指定列表以‘^’字符开始，那么指定的签名算法将放在默认集合的前面。此选项的默认设置为：

- ssh-ed25519-cert-v01@openssh.com,
- ecdsa-sha2-nistp256-cert-v01@openssh.com,
- ecdsa-sha2-nistp384-cert-v01@openssh.com,
- ecdsa-sha2-nistp521-cert-v01@openssh.com,
- sk-ssh-ed25519-cert-v01@openssh.com,
- sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
- rsa-sha2-512-cert-v01@openssh.com,
- rsa-sha2-256-cert-v01@openssh.com,
- ssh-ed25519,
- ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
- sk-ssh-ed25519@openssh.com,
- sk-ecdsa-sha2-nistp256@openssh.com,
- rsa-sha2-512,rsa-sha2-256

The -Q option of ssh(1) may be used to list supported signature algorithms. This was formerly named HostbasedKeyTypes.

可以使用 ssh 的 -Q 选项列出支持的签名算法。之前这个选项称为 HostbasedKeyTypes。

### HostbasedAuthentication

Specifies whether to try rhosts based authentication with public key authentication. The argument must be yes or no (the default).

指定是否尝试使用公钥认证进行rhosts基础认证。参数必须为yes或no（默认）。

### HostKeyAlgorithms

Specifies the host key signature algorithms that the client wants to use in order of preference. Alternately if the specified list begins with a ‘+’ character, then the specified signature algorithms will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified signature algorithms (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified signature algorithms will be placed at the head of the default set. The default for this option is:

指定客户端优先使用的主机密钥签名算法的顺序。或者，如果指定列表以‘+’字符开头，则指定的签名算法将附加到默认集合，而不是替换它们。如果指定列表以‘-’字符开头，则指定的签名算法（包括通配符）将从默认集合中移除，而不是替换它们。如果指定列表以‘^’字符开头，则指定的签名算法将置于默认集合的开头。该选项的默认值为：

- ssh-ed25519-cert-v01@openssh.com,
- ecdsa-sha2-nistp256-cert-v01@openssh.com,
- ecdsa-sha2-nistp384-cert-v01@openssh.com,
- ecdsa-sha2-nistp521-cert-v01@openssh.com,
- sk-ssh-ed25519-cert-v01@openssh.com,
- sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
- rsa-sha2-512-cert-v01@openssh.com,
- rsa-sha2-256-cert-v01@openssh.com,
- ssh-ed25519,
- ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
- sk-ecdsa-sha2-nistp256@openssh.com,
- sk-ssh-ed25519@openssh.com,
- rsa-sha2-512,rsa-sha2-256

If hostkeys are known for the destination host then this default is modified to prefer their algorithms.

如果已知目标主机的主机密钥，则默认设置会修改为优先使用这些算法。

The list of available signature algorithms may also be obtained using "ssh -Q HostKeyAlgorithms".

可用签名算法列表也可以通过"ssh -Q HostKeyAlgorithms"获取。

### HostKeyAlias

Specifies an alias that should be used instead of the real host name when looking up or saving the host key in the host key database files and when validating host certificates. This option is useful for tunneling SSH connections or for multiple servers running on a single host.

指定一个别名，用于在查找或保存主机密钥至主机密钥数据库文件以及验证主机证书时代替实际主机名。该选项对于SSH连接的隧道或单个主机上运行的多个服务器非常有用。

### Hostname

Specifies the real host name to log into. This can be used to specify nicknames or abbreviations for hosts. Arguments to Hostname accept the tokens described in the TOKENS section. Numeric IP addresses are also permitted (both on the command line and in Hostname specifications). The default is the name given on the command line.

指定要登录的实际主机名。可用于指定主机的昵称或缩写。Hostname 的参数接受 [TOKENS](#token) 部分中描述的标记。数字IP地址也允许用于命令行和主机名规范。默认值为命令行中给出的名称。

### IdentitiesOnly

Specifies that ssh(1) should only use the configured authentication identity and certificate files (either the default files, or those explicitly configured in the ssh_config files or passed on the ssh(1) command-line), even if ssh-agent(1) or a PKCS11Provider or SecurityKeyProvider offers more identities. The argument to this keyword must be yes or no (the default). This option is intended for situations where ssh-agent offers many different identities.

指定ssh只使用配置的认证身份和证书文件（默认文件或在ssh_config文件中显式配置的文件，或在ssh命令行传递的文件），即使ssh-agent或PKCS11Provider或SecurityKeyProvider提供更多身份。此关键词的参数必须是yes或no（默认）。此选项适用于ssh-agent提供多种身份的情况。

### IdentityAgent
Specifies the UNIX-domain socket used to communicate with the authentication agent.

This option overrides the SSH_AUTH_SOCK environment variable and can be used to select a specific agent. Setting the socket name to none disables the use of an authentication agent. If the string "SSH_AUTH_SOCK" is specified, the location of the socket will be read from the SSH_AUTH_SOCK environment variable. Otherwise if the specified value begins with a ‘$’ character, then it will be treated as an environment variable containing the location of the socket.

Arguments to IdentityAgent may use the tilde syntax to refer to a user's home directory, the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section.

### IdentityFile
Specifies a file from which the user's ECDSA, authenticator-hosted ECDSA, Ed25519, authenticator-hosted Ed25519 or RSA authentication identity is read. You can also specify a public key file to use the corresponding private key that is loaded in ssh-agent(1) when the private key file is not present locally. The default is ~/.ssh/id_rsa, ~/.ssh/id_ecdsa, ~/.ssh/id_ecdsa_sk, ~/.ssh/id_ed25519 and ~/.ssh/id_ed25519_sk. Additionally, any identities represented by the authentication agent will be used for authentication unless IdentitiesOnly is set. If no certificates have been explicitly specified by CertificateFile, ssh(1) will try to load certificate information from the filename obtained by appending -cert.pub to the path of a specified IdentityFile.

Arguments to IdentityFile may use the tilde syntax to refer to a user's home directory or the tokens described in the TOKENS section. Alternately an argument of none may be used to indicate no identity files should be loaded.

It is possible to have multiple identity files specified in configuration files; all these identities will be tried in sequence. Multiple IdentityFile directives will add to the list of identities tried (this behaviour differs from that of other configuration directives).

IdentityFile may be used in conjunction with IdentitiesOnly to select which identities in an agent are offered during authentication. IdentityFile may also be used in conjunction with CertificateFile in order to provide any certificate also needed for authentication with the identity.

### IgnoreUnknown
Specifies a pattern-list of unknown options to be ignored if they are encountered in configuration parsing. This may be used to suppress errors if ssh_config contains options that are unrecognised by ssh(1). It is recommended that IgnoreUnknown be listed early in the configuration file as it will not be applied to unknown options that appear before it.

### Include
Include the specified configuration file(s). Multiple pathnames may be specified and each pathname may contain glob(7) wildcards, tokens as described in the TOKENS section, environment variables as described in the ENVIRONMENT VARIABLES section and, for user configurations, shell-like ‘~’ references to user home directories. Wildcards will be expanded and processed in lexical order. Files without absolute paths are assumed to be in ~/.ssh if included in a user configuration file or /etc/ssh if included from the system configuration file. Include directive may appear inside a Match or Host block to perform conditional inclusion.

### IPQoS
Specifies the IPv4 type-of-service or DSCP class for connections. Accepted values are af11, af12, af13, af21, af22, af23, af31, af32, af33, af41, af42, af43, cs0, cs1, cs2, cs3, cs4, cs5, cs6, cs7, ef, le, lowdelay, throughput, reliability, a numeric value, or none to use the operating system default. This option may take one or two arguments, separated by whitespace. If one argument is specified, it is used as the packet class unconditionally. If two values are specified, the first is automatically selected for interactive sessions and the second for non-interactive sessions. The default is af21 (Low-Latency Data) for interactive sessions and cs1 (Lower Effort) for non-interactive sessions.

### KbdInteractiveAuthentication
Specifies whether to use keyboard-interactive authentication. The argument to this keyword must be yes (the default) or no. ChallengeResponseAuthentication is a deprecated alias for this.

### KbdInteractiveDevices
Specifies the list of methods to use in keyboard-interactive authentication. Multiple method names must be comma-separated. The default is to use the server specified list. The methods available vary depending on what the server supports. For an OpenSSH server, it may be zero or more of: bsdauth, pam, and skey.

### KexAlgorithms
Specifies the permitted KEX (Key Exchange) algorithms that will be used and their preference order. The selected algorithm will be the first algorithm in this list that the server also supports. Multiple algorithms must be comma-separated.
If the specified list begins with a ‘+’ character, then the specified algorithms will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified algorithms (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified algorithms will be placed at the head of the default set.

The default is:

sntrup761x25519-sha512,sntrup761x25519-sha512@openssh.com,
mlkem768x25519-sha256,
curve25519-sha256,curve25519-sha256@libssh.org,
ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,
diffie-hellman-group-exchange-sha256,
diffie-hellman-group16-sha512,
diffie-hellman-group18-sha512,
diffie-hellman-group14-sha256
The list of supported key exchange algorithms may also be obtained using "ssh -Q kex".

### KnownHostsCommand
Specifies a command to use to obtain a list of host keys, in addition to those listed in UserKnownHostsFile and GlobalKnownHostsFile. This command is executed after the files have been read. It may write host key lines to standard output in identical format to the usual files (described in the VERIFYING HOST KEYS section in ssh(1)). Arguments to KnownHostsCommand accept the tokens described in the TOKENS section. The command may be invoked multiple times per connection: once when preparing the preference list of host key algorithms to use, again to obtain the host key for the requested host name and, if CheckHostIP is enabled, one more time to obtain the host key matching the server's address. If the command exits abnormally or returns a non-zero exit status then the connection is terminated.

### LocalCommand
Specifies a command to execute on the local machine after successfully connecting to the server. The command string extends to the end of the line, and is executed with the user's shell. Arguments to LocalCommand accept the tokens described in the TOKENS section.
The command is run synchronously and does not have access to the session of the ssh(1) that spawned it. It should not be used for interactive commands.

This directive is ignored unless PermitLocalCommand has been enabled.

### LocalForward
Specifies that a TCP port on the local machine be forwarded over the secure channel to the specified host and port from the remote machine. The first argument specifies the listener and may be [bind_address:]port or a Unix domain socket path. The second argument is the destination and may be host:hostport or a Unix domain socket path if the remote host supports it.
IPv6 addresses can be specified by enclosing addresses in square brackets. Multiple forwardings may be specified, and additional forwardings can be given on the command line. Only the superuser can forward privileged ports. By default, the local port is bound in accordance with the GatewayPorts setting. However, an explicit bind_address may be used to bind the connection to a specific address. The bind_address of localhost indicates that the listening port be bound for local use only, while an empty address or ‘*’ indicates that the port should be available from all interfaces. Unix domain socket paths may use the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section.

### LogLevel
Gives the verbosity level that is used when logging messages from ssh(1). The possible values are: QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG, DEBUG1, DEBUG2, and DEBUG3. The default is INFO. DEBUG and DEBUG1 are equivalent. DEBUG2 and DEBUG3 each specify higher levels of verbose output.

### LogVerbose
Specify one or more overrides to LogLevel. An override consists of one or more pattern lists that matches the source file, function and line number to force detailed logging for. For example, an override pattern of:
kex.c:*:1000,*:kex_exchange_identification():*,packet.c:*
would enable detailed logging for line 1000 of kex.c, everything in the kex_exchange_identification() function, and all code in the packet.c file. This option is intended for debugging and no overrides are enabled by default.

### MACs
Specifies the MAC (message authentication code) algorithms in order of preference. The MAC algorithm is used for data integrity protection. Multiple algorithms must be comma-separated. If the specified list begins with a ‘+’ character, then the specified algorithms will be appended to the default set instead of replacing them. If the specified list begins with a ‘-’ character, then the specified algorithms (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified algorithms will be placed at the head of the default set.
The algorithms that contain "-etm" calculate the MAC after encryption (encrypt-then-mac). These are considered safer and their use recommended.

The default is:

umac-64-etm@openssh.com,umac-128-etm@openssh.com,
hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,
hmac-sha1-etm@openssh.com,
umac-64@openssh.com,umac-128@openssh.com,
hmac-sha2-256,hmac-sha2-512,hmac-sha1
The list of available MAC algorithms may also be obtained using "ssh -Q mac".

### NoHostAuthenticationForLocalhost
Disable host authentication for localhost (loopback addresses). The argument to this keyword must be yes or no (the default).

### NumberOfPasswordPrompts
Specifies the number of password prompts before giving up. The argument to this keyword must be an integer. The default is 3.

### ObscureKeystrokeTiming
Specifies whether ssh(1) should try to obscure inter-keystroke timings from passive observers of network traffic. If enabled, then for interactive sessions, ssh(1) will send keystrokes at fixed intervals of a few tens of milliseconds and will send fake keystroke packets for some time after typing ceases. The argument to this keyword must be yes, no or an interval specifier of the form interval:milliseconds (e.g. interval:80 for 80 milliseconds). The default is to obscure keystrokes using a 20ms packet interval. Note that smaller intervals will result in higher fake keystroke packet rates.

### PasswordAuthentication
Specifies whether to use password authentication. The argument to this keyword must be yes (the default) or no.
PermitLocalCommand
Allow local command execution via the LocalCommand option or using the !command escape sequence in ssh(1). The argument must be yes or no (the default).

### PermitRemoteOpen
Specifies the destinations to which remote TCP port forwarding is permitted when RemoteForward is used as a SOCKS proxy. The forwarding specification must be one of the following forms:
PermitRemoteOpen host:port
PermitRemoteOpen IPv4_addr:port
PermitRemoteOpen [IPv6_addr]:port
Multiple forwards may be specified by separating them with whitespace. An argument of any can be used to remove all restrictions and permit any forwarding requests. An argument of none can be used to prohibit all forwarding requests. The wildcard ‘*’ can be used for host or port to allow all hosts or ports respectively. Otherwise, no pattern matching or address lookups are performed on supplied names.

### PKCS11Provider
Specifies which PKCS#11 provider to use or none to indicate that no provider should be used (the default). The argument to this keyword is a path to the PKCS#11 shared library ssh(1) should use to communicate with a PKCS#11 token providing keys for user authentication.

### Port
Specifies the port number to connect on the remote host. The default is 22.
PreferredAuthentications
Specifies the order in which the client should try authentication methods. This allows a client to prefer one method (e.g. keyboard-interactive) over another method (e.g. password). The default is:
gssapi-with-mic,hostbased,publickey,
keyboard-interactive,password

### ProxyCommand
Specifies the command to use to connect to the server. The command string extends to the end of the line, and is executed using the user's shell ‘exec’ directive to avoid a lingering shell process.
Arguments to ProxyCommand accept the tokens described in the TOKENS section. The command can be basically anything, and should read from its standard input and write to its standard output. It should eventually connect an sshd(8) server running on some machine, or execute sshd -i somewhere. Host key management will be done using the Hostname of the host being connected (defaulting to the name typed by the user). Setting the command to none disables this option entirely. Note that CheckHostIP is not available for connects with a proxy command.

This directive is useful in conjunction with nc(1) and its proxy support. For example, the following directive would connect via an HTTP proxy at 192.0.2.0:

ProxyCommand /usr/bin/nc -X connect -x 192.0.2.0:8080 %h %p

### ProxyJump
Specifies one or more jump proxies as either [user@]host[:port] or an ssh URI. Multiple proxies may be separated by comma characters and will be visited sequentially. Setting this option will cause ssh(1) to connect to the target host by first making a ssh(1) connection to the specified ProxyJump host and then establishing a TCP forwarding to the ultimate target from there. Setting the host to none disables this option entirely.
Note that this option will compete with the ProxyCommand option - whichever is specified first will prevent later instances of the other from taking effect.

Note also that the configuration for the destination host (either supplied via the command-line or the configuration file) is not generally applied to jump hosts. ~/.ssh/config should be used if specific configuration is required for jump hosts.

### ProxyUseFdpass
Specifies that ProxyCommand will pass a connected file descriptor back to ssh(1) instead of continuing to execute and pass data. The default is no.

### PubkeyAcceptedAlgorithms
Specifies the signature algorithms that will be used for public key authentication as a comma-separated list of patterns. If the specified list begins with a ‘+’ character, then the algorithms after it will be appended to the default instead of replacing it. If the specified list begins with a ‘-’ character, then the specified algorithms (including wildcards) will be removed from the default set instead of replacing them. If the specified list begins with a ‘^’ character, then the specified algorithms will be placed at the head of the default set. The default for this option is:
ssh-ed25519-cert-v01@openssh.com,
ecdsa-sha2-nistp256-cert-v01@openssh.com,
ecdsa-sha2-nistp384-cert-v01@openssh.com,
ecdsa-sha2-nistp521-cert-v01@openssh.com,
sk-ssh-ed25519-cert-v01@openssh.com,
sk-ecdsa-sha2-nistp256-cert-v01@openssh.com,
rsa-sha2-512-cert-v01@openssh.com,
rsa-sha2-256-cert-v01@openssh.com,
ssh-ed25519,
ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
sk-ssh-ed25519@openssh.com,
sk-ecdsa-sha2-nistp256@openssh.com,
rsa-sha2-512,rsa-sha2-256
The list of available signature algorithms may also be obtained using "ssh -Q PubkeyAcceptedAlgorithms".

### PubkeyAuthentication
Specifies whether to try public key authentication. The argument to this keyword must be yes (the default), no, unbound or host-bound. The final two options enable public key authentication while respectively disabling or enabling the OpenSSH host-bound authentication protocol extension required for restricted ssh-agent(1) forwarding.

### RekeyLimit
Specifies the maximum amount of data that may be transmitted or received before the session key is renegotiated, optionally followed by a maximum amount of time that may pass before the session key is renegotiated. The first argument is specified in bytes and may have a suffix of ‘K’, ‘M’, or ‘G’ to indicate Kilobytes, Megabytes, or Gigabytes, respectively. The default is between ‘1G’ and ‘4G’, depending on the cipher. The optional second value is specified in seconds and may use any of the units documented in the TIME FORMATS section of sshd_config(5). The default value for RekeyLimit is default none, which means that rekeying is performed after the cipher's default amount of data has been sent or received and no time based rekeying is done.

### RemoteCommand
Specifies a command to execute on the remote machine after successfully connecting to the server. The command string extends to the end of the line, and is executed with the user's shell. Arguments to RemoteCommand accept the tokens described in the TOKENS section.

### RemoteForward
Specifies that a TCP port on the remote machine be forwarded over the secure channel. The remote port may either be forwarded to a specified host and port from the local machine, or may act as a SOCKS 4/5 proxy that allows a remote client to connect to arbitrary destinations from the local machine. The first argument is the listening specification and may be [bind_address:]port or, if the remote host supports it, a Unix domain socket path. If forwarding to a specific destination then the second argument must be host:hostport or a Unix domain socket path, otherwise if no destination argument is specified then the remote forwarding will be established as a SOCKS proxy. When acting as a SOCKS proxy, the destination of the connection can be restricted by PermitRemoteOpen.
IPv6 addresses can be specified by enclosing addresses in square brackets. Multiple forwardings may be specified, and additional forwardings can be given on the command line. Privileged ports can be forwarded only when logging in as root on the remote machine. Unix domain socket paths may use the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section.

If the port argument is 0, the listen port will be dynamically allocated on the server and reported to the client at run time.

If the bind_address is not specified, the default is to only bind to loopback addresses. If the bind_address is ‘*’ or an empty string, then the forwarding is requested to listen on all interfaces. Specifying a remote bind_address will only succeed if the server's GatewayPorts option is enabled (see sshd_config(5)).

### RequestTTY
Specifies whether to request a pseudo-tty for the session. The argument may be one of: no (never request a TTY), yes (always request a TTY when standard input is a TTY), force (always request a TTY) or auto (request a TTY when opening a login session). This option mirrors the -t and -T flags for ssh(1).

### RequiredRSASize
Specifies the minimum RSA key size (in bits) that ssh(1) will accept. User authentication keys smaller than this limit will be ignored. Servers that present host keys smaller than this limit will cause the connection to be terminated. The default is 1024 bits. Note that this limit may only be raised from the default.

### RevokedHostKeys
Specifies revoked host public keys. Keys listed in this file will be refused for host authentication. Note that if this file does not exist or is not readable, then host authentication will be refused for all hosts. Keys may be specified as a text file, listing one public key per line, or as an OpenSSH Key Revocation List (KRL) as generated by ssh-keygen(1). For more information on KRLs, see the KEY REVOCATION LISTS section in ssh-keygen(1). Arguments to RevokedHostKeys may use the tilde syntax to refer to a user's home directory, the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section.

### SecurityKeyProvider
Specifies a path to a library that will be used when loading any FIDO authenticator-hosted keys, overriding the default of using the built-in USB HID support.
If the specified value begins with a ‘$’ character, then it will be treated as an environment variable containing the path to the library.

### SendEnv
Specifies what variables from the local environ(7) should be sent to the server. The server must also support it, and the server must be configured to accept these environment variables. Note that the TERM environment variable is always sent whenever a pseudo-terminal is requested as it is required by the protocol. Refer to AcceptEnv in sshd_config(5) for how to configure the server. Variables are specified by name, which may contain wildcard characters. Multiple environment variables may be separated by whitespace or spread across multiple SendEnv directives.
See PATTERNS for more information on patterns.

It is possible to clear previously set SendEnv variable names by prefixing patterns with -. The default is not to send any environment variables.

### ServerAliveCountMax
Sets the number of server alive messages (see below) which may be sent without ssh(1) receiving any messages back from the server. If this threshold is reached while server alive messages are being sent, ssh will disconnect from the server, terminating the session. It is important to note that the use of server alive messages is very different from TCPKeepAlive (below). The server alive messages are sent through the encrypted channel and therefore will not be spoofable. The TCP keepalive option enabled by TCPKeepAlive is spoofable. The server alive mechanism is valuable when the client or server depend on knowing when a connection has become unresponsive.
The default value is 3. If, for example, ServerAliveInterval (see below) is set to 15 and ServerAliveCountMax is left at the default, if the server becomes unresponsive, ssh will disconnect after approximately 45 seconds.

### ServerAliveInterval
Sets a timeout interval in seconds after which if no data has been received from the server, ssh(1) will send a message through the encrypted channel to request a response from the server. The default is 0, indicating that these messages will not be sent to the server.

### SessionType
May be used to either request invocation of a subsystem on the remote system, or to prevent the execution of a remote command at all. The latter is useful for just forwarding ports. The argument to this keyword must be none (same as the -N option), subsystem (same as the -s option) or default (shell or command execution).

### SetEnv
Directly specify one or more environment variables and their contents to be sent to the server. Similarly to SendEnv, with the exception of the TERM variable, the server must be prepared to accept the environment variable.

### StdinNull
Redirects stdin from /dev/null (actually, prevents reading from stdin). Either this or the equivalent -n option must be used when ssh is run in the background. The argument to this keyword must be yes (same as the -n option) or no (the default).

### StreamLocalBindMask
Sets the octal file creation mode mask (umask) used when creating a Unix-domain socket file for local or remote port forwarding. This option is only used for port forwarding to a Unix-domain socket file.
The default value is 0177, which creates a Unix-domain socket file that is readable and writable only by the owner. Note that not all operating systems honor the file mode on Unix-domain socket files.

### StreamLocalBindUnlink
Specifies whether to remove an existing Unix-domain socket file for local or remote port forwarding before creating a new one. If the socket file already exists and StreamLocalBindUnlink is not enabled, ssh will be unable to forward the port to the Unix-domain socket file. This option is only used for port forwarding to a Unix-domain socket file.
The argument must be yes or no (the default).

### StrictHostKeyChecking
If this flag is set to yes, ssh(1) will never automatically add host keys to the ~/.ssh/known_hosts file, and refuses to connect to hosts whose host key has changed. This provides maximum protection against man-in-the-middle (MITM) attacks, though it can be annoying when the /etc/ssh/ssh_known_hosts file is poorly maintained or when connections to new hosts are frequently made. This option forces the user to manually add all new hosts.
If this flag is set to accept-new then ssh will automatically add new host keys to the user's known_hosts file, but will not permit connections to hosts with changed host keys. If this flag is set to no or off, ssh will automatically add new host keys to the user known hosts files and allow connections to hosts with changed hostkeys to proceed, subject to some restrictions. If this flag is set to ask (the default), new host keys will be added to the user known host files only after the user has confirmed that is what they really want to do, and ssh will refuse to connect to hosts whose host key has changed. The host keys of known hosts will be verified automatically in all cases.

### SyslogFacility
Gives the facility code that is used when logging messages from ssh(1). The possible values are: DAEMON, USER, AUTH, LOCAL0, LOCAL1, LOCAL2, LOCAL3, LOCAL4, LOCAL5, LOCAL6, LOCAL7. The default is USER.

### TCPKeepAlive
Specifies whether the system should send TCP keepalive messages to the other side. If they are sent, death of the connection or crash of one of the machines will be properly noticed. However, this means that connections will die if the route is down temporarily, and some people find it annoying.
The default is yes (to send TCP keepalive messages), and the client will notice if the network goes down or the remote host dies. This is important in scripts, and many users want it too.

To disable TCP keepalive messages, the value should be set to no. See also ServerAliveInterval for protocol-level keepalives.

### Tag
Specify a configuration tag name that may be later used by a Match directive to select a block of configuration.

### Tunnel
Request tun(4) device forwarding between the client and the server. The argument must be yes, point-to-point (layer 3), ethernet (layer 2), or no (the default). Specifying yes requests the default tunnel mode, which is point-to-point.

### TunnelDevice
Specifies the tun(4) devices to open on the client (local_tun) and the server (remote_tun).
The argument must be local_tun[:remote_tun]. The devices may be specified by numerical ID or the keyword any, which uses the next available tunnel device. If remote_tun is not specified, it defaults to any. The default is any:any.

### UpdateHostKeys
Specifies whether ssh(1) should accept notifications of additional hostkeys from the server sent after authentication has completed and add them to UserKnownHostsFile. The argument must be yes, no or ask. This option allows learning alternate hostkeys for a server and supports graceful key rotation by allowing a server to send replacement public keys before old ones are removed.
Additional hostkeys are only accepted if the key used to authenticate the host was already trusted or explicitly accepted by the user, the host was authenticated via UserKnownHostsFile (i.e. not GlobalKnownHostsFile) and the host was authenticated using a plain key and not a certificate.

UpdateHostKeys is enabled by default if the user has not overridden the default UserKnownHostsFile setting and has not enabled VerifyHostKeyDNS, otherwise UpdateHostKeys will be set to no.

If UpdateHostKeys is set to ask, then the user is asked to confirm the modifications to the known_hosts file. Confirmation is currently incompatible with ControlPersist, and will be disabled if it is enabled.

Presently, only sshd(8) from OpenSSH 6.8 and greater support the "hostkeys@openssh.com" protocol extension used to inform the client of all the server's hostkeys.

### User
Specifies the user to log in as. This can be useful when a different user name is used on different machines. This saves the trouble of having to remember to give the user name on the command line.

### UserKnownHostsFile
Specifies one or more files to use for the user host key database, separated by whitespace. Each filename may use tilde notation to refer to the user's home directory, the tokens described in the TOKENS section and environment variables as described in the ENVIRONMENT VARIABLES section. A value of none causes ssh(1) to ignore any user-specific known hosts files. The default is ~/.ssh/known_hosts, ~/.ssh/known_hosts2.

### VerifyHostKeyDNS
Specifies whether to verify the remote key using DNS and SSHFP resource records. If this option is set to yes, the client will implicitly trust keys that match a secure fingerprint from DNS. Insecure fingerprints will be handled as if this option was set to ask. If this option is set to ask, information on fingerprint match will be displayed, but the user will still need to confirm new host keys according to the StrictHostKeyChecking option. The default is no.
See also VERIFYING HOST KEYS in ssh(1).

### VisualHostKey
If this flag is set to yes, an ASCII art representation of the remote host key fingerprint is printed in addition to the fingerprint string at login and for unknown host keys. If this flag is set to no (the default), no fingerprint strings are printed at login and only the fingerprint string will be printed for unknown host keys.

### XAuthLocation
Specifies the full pathname of the xauth(1) program. The default is /usr/X11R6/bin/xauth.

## 模式（PATTERNS）

## TOKEN


