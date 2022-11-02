# 常用代理设置

## CMD

```shell
# 设置
set http_proxy=http://127.0.0.1:1080
set https_proxy=http://127.0.0.1:1080
# 取消
set http_proxy=
set https_proxy=
```

## PowerShell

```powershell
$env:http_proxy="127.0.0.1:1080"
$env:https_proxy="127.0.0.1:1080"
```

为了方便，将下面函数添加到 `$PROFILE` 中就可以通过 proxy 和 unproxy 来实现设置与取消代理了。

```powershell
# Set and unset proxy for PowerShell
function proxy {
    param (
        $ssr = "127.0.0.1:1080"
    )
    New-Item -Path Env: -Name http_proxy -Value $ssr
    New-Item -Path Env: -Name https_proxy -Value $ssr
}

function unproxy {
    Remove-Item -Path Env:\http_proxy
    Remove-Item -Path Env:\https_proxy
}
```

## Git Bash

```bash
# 设置
export http_proxy=http://127.0.0.1:1080
export https_proxy=http://127.0.0.1:1080
# 取消
unset http_proxy https_proxy
```

## All APPs

```shell
# 设置
netsh winhttp import proxy source=ie
# 取消
netsh winhttp reset proxy
```

## 为 Git 设置代理

众所周知，`git clone` 有两种方式，代理设置方式也不一样：

### Clone with HTTPS

设置代理：

```shell
# 如果是 socks5 代理的话
git config --global http.proxy socks5h://127.0.0.1:1080
# http 代理仅需将 socks5h 改为 http
git config --global http.proxy http://127.0.0.1:1080
```

取消代理：

```shell
git config --global --unset http.proxy
```

也可以仅为 GitHub 设置代理

```shell
git config --global http.https://github.com.proxy socks5h://127.0.0.1:1080
```

socks5h 和 socks5 的区别：

> In a proxy string, socks5h:// and socks4a:// mean that the hostname is
resolved by the SOCKS server. socks5:// and socks4:// mean that the
hostname is resolved locally. socks4a:// means to use SOCKS4a, which is
an extension of SOCKS4.

来源：[Differentiate socks5h from socks5 and socks4a from socks4 when handling proxy string](https://github.com/urllib3/urllib3/issues/1035)

### Clone with SSH

需要修改 `~/.ssh/config` 文件

如果仅为 GitHub 设置代理，且使用 socks5 代理的话

```text
Host github.com
    User git
    ProxyCommand connect -S 127.0.0.1:1080 %h %p
```

这里 `-S` 表示使用 socks5 代理，如果是 http 代理则为 `-H`。connect 工具 [Git for Windows](https://gitforwindows.org) 自带。

我自己的话，则是设置成这样：

```text
# Reference: https://bitbucket.org/gotoh/connect/wiki/Home

Host github.com
    User git
    Hostname github.com
    Port 22
    ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p

Host ssh.github.com
    User git
    Hostname ssh.github.com
    Port 443
    ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p

Host *
    IdentityFile "C:\Users\Zheng\.ssh\id_rsa"
    ServerAliveInterval 30
    TCPKeepAlive yes
```

来源：[laispace/git 设置和取消代理](https://gist.github.com/laispace/666dd7b27e9116faece6)

## SSH 加速

### 通过代理连接

其实就是借助 [ProxyCommand](https://man.openbsd.org/ssh_config.5#ProxyCommand) 这个选项来实现的，并且有几种不同的写法。而且，Windows 和 macOS 下的实现方式也不一样。

假定本地代理地址为 `127.0.0.1`，端口为 `1080`，代理方式为 `socks5`，要连接的远程主机用户为 `root`，主机 IP 为 `1.1.1.1`。

#### Windows - connect

如果下面无效的话请先安装 [connect](https://web.archive.org/web/20080516100455/http://www.meadowy.org/~gotoh/projects/connect) 这个小工具并将其添加至环境变量，或者直接在 Git Bash 中操作。

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
Host github.com
    User git
    Hostname github.com
    Port 22
    ProxyCommand connect -S 127.0.0.1:1080 -a none %h %p
```

#### macOS - netcat

macOS 下可以直接看这篇[文章](https://www.xiebruce.top/650.html#i)。

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

<mark>注意</mark>：`%h` 和 `%p` 之间是 `:` 而不是空格。

```shell
ssh -o "ProxyCommand ssh root@8.8.8.8 -W %h:%p" root@1.1.1.1
```

- 写法二

使用这条命令，必须先安装 [netcat](https://eternallybored.org/misc/netcat/) 并将其添加至环境变量。

```shell
ssh -o "ProxyCommand ssh root@8.8.8.8 nc %h %p" root@1.1.1.1
```

#### 写入配置文件

即写入到 `~/.ssh/config` 文件中，这个我目前没有需求，同样可以查看这篇[文章](https://www.xiebruce.top/650.html#i-9)获取详细内容。
