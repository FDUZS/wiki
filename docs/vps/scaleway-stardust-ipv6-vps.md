# Scaleway Stardust IPv6 VPS

纯 IPv6 的 Stardust 非常便宜，一年不到 6 欧元，作为玩具还是可以的。

## 创建账号

在官网创建账号，应该还需要添加信用卡才能使用，会赠送一定的体验金，记得查收邮箱并兑换。

## 添加 SSH 密钥

参照 SSH 文档生成密钥，并在后台添加。

## 利用 Scaleway CLI 创建机器

### 安装 Scaleway CLI

参考 [Scaleway CLI] 文档进行安装，比如 `scoop install scaleway-cli`

### 生成 API Keys

到 [Identity and Access Management (IAM)] 页面生成，并复制 `secret key`

### 创建机器

先通过 `scw init` 进行初始化并生成配置文件，如图

![scw-init](https://img.shuaizheng.net/2307/scw-init.png)

通过以下命令创建服务器

```shell
scw instance server create type=STARDUST1-S zone=fr-par-1 image=debian_bullseye root-volume=l:10G ip=none ipv6=true project-id=xxx
```

- `type=STARDUST1-S` 表示创建 Stardust
- `zone=fr-par-1` 表示区域为法国巴黎一区
- `image=debian_bullseye` 表示系统为 Debian 11
- `root-volume=l:10G` 表示硬盘为 10G
- `ip=none ipv6=true` 表示仅 IPv6
- ==将 project-id=xxx 中的 xxx 替换为初始化时的 default_project_id==

## 利用 WARP 添加 IPv4 网络

[WARP] 是 Cloudflare 推出的基于 WireGuard 的 VPN

==以下内容基于 Debian 10+==

### 安装 WireGuard

准备工作，安装 `sudo` 和 `lsb_release`

```shell
apt install sudo lsb-release -y
```

安装必要的网络工具

```shell
sudo apt install net-tools iproute2 openresolv dnsutils -y
```

安装 Wire­Guard 配置工具

```shell
sudo apt install wireguard-tools --no-install-recommends
```

通过 `uname -r` 查看内核版本，==如果是 5.6 及以上内核则已经集成了 Wire­Guard，就不需要安装了==。

否则，需要先添加 back­ports 源

```shell
echo "deb http://deb.debian.org/debian $(lsb_release -sc)-backports main" | sudo tee /etc/apt/sources.list.d/backports.list
sudo apt update
```

再安装 back­ports 仓库中的内核

```shell
sudo apt -t $(lsb_release -sc)-backports install linux-image-$(dpkg --print-architecture) linux-headers-$(dpkg --print-architecture) --install-recommends -y
```

安装完成后，再次执行 `uname -r` 确保新版内核已启用

### 通过 wgcf 生成配置文件

在安装之前，先修改 DNS 以便下面操作，将 `/etc/resolv.conf` 中的 nameserver 修改为以下

```text
nameserver 2a03:7900:2:0:31:3:104:161
nameserver 2a00:1098:2b::1
nameserver 2a01:4f8:c2c:123f::1
nameserver 2a00:1098:2c::1
```

安装 wgcf

```shell
curl -fsSL git.io/wgcf.sh | sudo bash
```

注册 WARP 账户

```shell
wgcf register
```

生成 Wire­Guard 配置文件

```shell
wgcf generate
```

### 编辑配置文件

修改 `wgcf-profile.conf` 配置文件

- 将 `engage.cloudflareclient.com` 替换为 `[2606:4700:d0::a29f:c001]`
- 删除或注释掉 `AllowedIPs = ::/0`
- 将 `DNS = 1.1.1.1` 修改为 `DNS = 2606:4700:4700::1111`

完成并保存后，将 Wire­Guard 配置文件复制到 `/etc/wireguard/` 并命名为 `wgcf.conf`

```shell
sudo cp wgcf-profile.conf /etc/wireguard/wgcf.conf
```

开启网络接口（命令中的 `wgcf` 对应的是配置文件 `wgcf.conf` 的文件名前缀）

```shell
sudo wg-quick up wgcf
```

使用 `curl -4 ip.sb` 看看能否顺利返回 IPv4

没问题后，执行 `crontab -e` 命令，添加 `@reboot systemctl start wg-quick@wgcf` 到文件末尾设置开机自启。

## 挂载对象存储

可选

## 美化终端

### 安装字体

安装 [Nerd Font]，推荐 Fira Code Nerd Font

在用户目录新建 `~/.fonts` 文件夹存放字体，进入该文件夹使用 `wget` 下载后解压

`sudo apt install fontconfig` 安装后使用 `fc-cache` 更新字体缓存

### 安装 Starship

```shell
curl -sS https://starship.rs/install.sh | sh
```

### 设置 shell

Bash 的话，在 `~/.bashrc` 末尾添加一行，其他的可查看 [Starship] 安装文档

```shell
eval "$(starship init bash)"
```

完工，断开重连后即可看到效果。

## 参考

- [API 创建 Scaleway Stardust IPv6 VPS]
- [Scaleway Stardust 纯 IPv6 服务器体验]
- [Debian Linux VPS 服务器 WireGuard 安装教程]
- [Cloudflare WARP 给 Linux VPS 云服务器添加原生 IPv4/IPv6 双栈网络]

[Scaleway CLI]: https://github.com/scaleway/scaleway-cli
[API 创建 Scaleway Stardust IPv6 VPS]: https://www.uionm.com/2022/11/945/
[Scaleway Stardust 纯 IPv6 服务器体验]: https://aoyouer.com/posts/scaleway-ipv6only-server/
[Debian Linux VPS 服务器 WireGuard 安装教程]: https://p3terx.com/archives/debian-linux-vps-server-wireguard-installation-tutorial.html
[Cloudflare WARP 给 Linux VPS 云服务器添加原生 IPv4/IPv6 双栈网络]: https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html
[Identity and Access Management (IAM)]: https://console.scaleway.com/iam/api-keys
[WARP]: https://blog.cloudflare.com/1111-warp-better-vpn/
[Nerd Font]: https://www.nerdfonts.com/
[Starship]: https://starship.rs/
