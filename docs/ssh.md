# SSH

当前用户配置文件目录：`~/.ssh`

其他介绍，略。

## SSH 密钥生成

进入配置文件目录

Ed25519（推荐）

将下面尖括号及内部替换成想要的。`-f` 参数指定生成的文件名，如 `vps_ed25519`，`-C` 参数起注释（comment）作用。下同。

```shell
ssh-keygen -t ed25519 -f <id>_ed25519 -C "<comment>"
```

RSA（仅推荐 2048 位以上）

```shell
ssh-keygen -t rsa -b 4096 -f <id>_rsa -C "<comment>"
```

## SSH 加速

### 通过代理连接

其实就是借助 [ProxyCommand] 这个选项来实现的，并且有几种不同的写法。而且，Windows 和 macOS 下的实现方式也不一样。

假定本地代理地址为 `127.0.0.1`，端口为 `1080`，代理方式为 `socks5`，要连接的远程主机用户为 `root`，主机 IP 为 `1.1.1.1`。

#### Windows - connect

如果下面无效的话请先安装 [connect] 这个小工具并将其添加至环境变量，或者直接在 Git Bash 中操作。

先解释一下参数含义：`connect` 即是上面安装的工具，`-S` 表示使用 socks5 代理，`-a none` 表示本地代理无需认证，`%h %p` 分别对应远程主机名和端口。

- 写法一

```shell
ssh -o "ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p" root@1.1.1.1
```

- 写法二

```shell
ssh -o ProxyCommand="connect -S 127.0.0.1:1080 -a none %h %p" root@1.1.1.1
```

- 写法三

写到 `~/.ssh/config` 文件中，如针对 GitHub 可以这样写：

```text
Host dev
    HostName 1.1.1.1
    # Port 22
    User root
    IdentityFile ~/.ssh/vps_ed25519
    ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p
```

#### macOS - netcat

macOS 下可以直接看这篇[文章][macOS]。

### 借助跳板机连接

太详细的我也懒得写了，目前只写一下 Windows 如何实现。ProxyJump 和 ProxyCommand 都是可以的，并且如果同时写在配置文件中，只能是最先匹配到的那个生效。

假定跳板机和真正要登录的远程主机用户都为 `root`，跳板机 IP 为 `8.8.8.8`，要登录的主机 IP 为 `1.1.1.1`。

各参数含义与第一部分一致，`-W` 表示将客户端的标准输入输出转发到相应端口的远程主机上。

#### ProxyJump

```shell
ssh -o "ProxyJump root@8.8.8.8" root@1.1.1.1
```

#### ProxyCommand

- 写法一

==注意==：`%h` 和 `%p` 之间是 `:` 而不是空格。

```shell
ssh -o "ProxyCommand ssh root@8.8.8.8 -W %h:%p" root@1.1.1.1
```

- 写法二

使用这条命令，必须先安装 [netcat] 并将其添加至环境变量。

```shell
ssh -o "ProxyCommand ssh root@8.8.8.8 nc %h %p" root@1.1.1.1
```

#### 写入配置文件

即写入到 `~/.ssh/config` 文件中。

先配置跳板机，如果需要通过代理和密钥连接，则配置如下

```text
Host jumpserver
    HostName 8.8.8.8
    # Port 22
    User root
    IdentityFile ~/.ssh/vps_ed25519
    ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p
```

使用 ProxyJump 配置

```text
Host dev
    HostName 1.1.1.1
    # Port 22
    User root
    IdentityFile ~/.ssh/vps_ed25519
    ProxyJump jumpserver
```

使用 ProxyCommand 配置

- 写法一

```text
Host dev
    HostName 1.1.1.1
    # Port 22
    User root
    IdentityFile ~/.ssh/vps_ed25519
    ProxyCommand ssh jumpserver -W %h:%p
```

- 写法二

```text
Host dev
    HostName 1.1.1.1
    # Port 22
    User root
    IdentityFile ~/.ssh/vps_ed25519
    ProxyCommand ssh jumpserver nc %h %p
```

[ProxyCommand]: https://man.openbsd.org/ssh_config.5#ProxyCommand
[connect]: https://web.archive.org/web/20080516100455/http://www.meadowy.org/~gotoh/projects/connect
[macOS]: https://www.xiebruce.top/650.html
[netcat]: https://eternallybored.org/misc/netcat/
