# 用 IPv6 来拯救你的校园网

### 购置 VPS

你用哪家都行，支持 IPv6 就好。

我在用的是 RamNode 家的，这是[邀请链接](https://clientarea.ramnode.com/aff.php?aff=3737)。不喜，可直接访问其[首页](https://www.ramnode.com/)。

顺便一提，他家现在更新面板了，而且采用计时收费的方式，随时可以销毁服务器。如果决定使用他家的 VPS Instance，~~建议系统选择为 Ubuntu 18.04 及以上而不要选 Debian 系列的，好像 Debian 系列有 Bug，IPv6 是不通的~~<mark>他家的 Bug 很诡异，自己在 Debian 9、Ubuntu 18、CentOS 7 中一个个尝试吧，反正按时计费</mark>。

### 服务端搭建

新手简单使用的话，直接看这个就好：[某大型同性交友网站指南](https://github.com/233boy/ss/wiki/Shadowsocks%E6%90%AD%E5%BB%BA%E8%AF%A6%E7%BB%86%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B)。

### 客户端安装

愿意尝试使用 [Scoop](https://www.iamzs.top/archives/scoop-guidebook.html) 的话，安装只需一条命令：`scoop install shadowsocksr-csharp`。

手动安装的话依然是：[某大型同性交友网站](https://github.com/shadowsocksrr/shadowsocksr-csharp/releases)。

### 客户端使用

如果是用上面脚本的话，你甚至可以通过「二维码扫描」自动填写相关服务信息。

此外，建议「系统代理模式」选`全局模式`，「PAC」选`更新 PAC 为 GFWList`，「代理规则」选`绕过局域网和大陆`。

### Proxifier 设置

可以看看这篇，他也是参考了我之前写的：[Phantom T](https://phantomt.github.io/2019/05/02/Linux-000-VPS/#%E4%BD%BF%E7%94%A8Proxifier%E8%BF%9B%E8%A1%8C%E5%85%A8%E5%B1%80%E4%BB%A3%E7%90%86)。

其他的应该就没有了。

<!-- 好好学习，爱国敬业。多思考，增强明辨是非的能力，不信谣不传谣。

外网优秀的学习资源那么多，如果只是看些捕风捉影、耸人听闻的假消息，实在是不应该。 -->
