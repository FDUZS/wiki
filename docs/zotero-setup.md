# Zotero 设置

恰逢 Zotero 6.0 正式版发布没多久，来填两年多前留的这个坑，应该不会太晚。主要讲一讲如何设置并且快速上手使用，而且根据我的观察，很多人对其中一些概念分得似乎不是很清，影响了他们的工作流程设定。

### 前置要求

在开始之前，先安装下面的插件和软件。当然，假定你已经安装了 [Zotero](https://www.zotero.org/download/) 最新版以及你所使用的浏览器对应的 Zotero Connector 插件。

#### 插件篇

- [ZotFile](http://zotfile.com/) 可以方便添加附件及重命名

- [Jasminum - 茉莉花](https://github.com/l0o0/jasminum) 对国内文献数据库（如：CNKI）支持良好，可自动下载文献 PDF

- [Better BibTeX](https://retorque.re/zotero-better-bibtex/) 可选，如果你不知道 LaTeX 是啥的话

- [Zotero Scihub](https://github.com/ethanwillis/zotero-scihub) 可帮助你尝试从 Sci-Hub 上下载文献

#### 软件篇

- [PDFtk Server](https://www.pdflabs.com/tools/pdftk-server/) 上面的茉莉花插件需要，可帮助自动生成和添加 PDF 书签（仅在支持的情况下，如国内硕博论文和一些外文期刊。下载硕博论文 PDF 版，可转到[知网海外版](https://chn.oversea.cnki.net/index/)）。你可通过 [Scoop](https://www.iamzs.top/archives/scoop-guidebook.html) 安装 `scoop install pdftk`。当然，也可手动[安装](https://www.pdflabs.com/tools/pdftk-server/)。由于我使用的是 Windows，如果你是其他系统的话，请到上面茉莉花页面查看对应安装说明。

#### 账号篇

- Zotero 账号，官网注册即可

- 支持 WebDAV 协议的云存储工具，如[坚果云](https://www.jianguoyun.com/)等，下文以此为例

- 任一你常用的云存储服务，如 OneDrive 等，下文以此为例

### 软件及插件设置

以下描述中英文参杂，我也懒得切换软件语言和配图，还请仔细查找对照设置项。以下选项属个人偏好，如果第一次使用的话，可完全根据我的来，之后再作调整。

不过到这里出现了两种路径，即只在桌面端使用 Zotero 和搭配 iPad 使用。在那之前，先介绍一下 Zotero，我们利用它保存的无非两种数据：文献条目信息及对应的附件，前者包含文献的元数据，方便写作时引用和著录，后者则是文献 PDF 和笔记等。Zotero 免费提供了 300MB 的存储空间，如果附件也一并同步的话肯定是不够的，当然你也可以年付 120 美金以获取无限空间，那样的话也不用纠结和折腾了。

好了，确定你我都是暂时无力负担官方存储服务之人，在附件同步的选择上只剩下链接形式和 WebDAV 这两种，与上面两种路径分别对应。使用前者同步的是纯 PDF 文件，使用后者的话，每一条文献条目会同步两个文件，一个是 PDF 的压缩文件，另一个我也不知道是啥，可参看[官方文档](https://www.zotero.org/support/sync)页面。

因此，如果不需要在平板上看文献的话，我不建议使用 WebDAV 同步。顺嘴说一句，团队类型的文献库即使是附件也只能通过官方同步，不过个人使用可以先不管。

#### 两种路径共有设置

**General**

- 取消勾选 Automatically take snapshots when creating items from web pages

  我不需要它自动创建文献的网页快照

- 取消勾选 Automatically rename attachment files using parent metadata

  使用 ZotFile 来重命名

- 取消勾选 Automatically tag items with keywords and subject headings

  不让软件自动打标签

**Cite**

- 添加国标引用和著录样式，留给你自己去探索

**茉莉花插件设置**

- 勾选 Retrieve CNKI meta data when add Chinese articles

- PDFtk Server Execute File Path 选择 PDFtk Server 安装目录下的 bin 目录

- 在 Unofficial Translators Repository 中更新所有解析器，并在 浏览器插件设置 Advanced - Translators 中点击 Update Translators

**Zotero Scihub 插件**

- 取消勾选自动下载 PDF，在新版本中它可能会打断浏览器插件和茉莉花插件的文献下载

- 根据实际情况修改 URL，如 https://sci-hub.se/

**ZotFile 插件设置**

- Source Folder for Attaching Files 设置为常用的下载文件夹

- Other Advanced Settings 选择 Always rename

#### 区别设置

=== "只在桌面端使用"

    **Sync**

    - 登录账号并取消勾选 Sync automatically，也可保持勾选

    **Advanced**

    - Base Directory 设置为 OneDrive PDF 文件夹，即你想放置和同步 PDF 文献的地方

    **ZotFile 插件设置**

    - Custom Location 设置为上述 Base Directory

=== "搭配 iPad 使用"

    **Sync**

    - 登录账号并在下方文件同步处将默认的 Zotero 改为 WebDAV，这是坚果云官方提供的[帮助文档](https://help.jianguoyun.com/?p=3168)

    - 在 iPad 上登录账号，并将同步方式改为 WebDAV，使用方式同上

### 结尾

至此，大致的工作流已经形成了。

采用 WebDAV 同步的同学，在数据库中看到一篇不错的文献，点击浏览器上插件的图标将自动为你抓取文献信息和尝试下载文献 PDF 并保存到 Zotero 上。否则你将需要手动下载保存到常用下载文件夹，然后回到 Zotero 右键点击该文献条目，选择 Attach New File，弹出对话框询问你是否将最近下载的那个文件添加为附件。然后，坚果云等工具工作，便可以在 iPad 上点击文献条目，下载文献，阅读。你在任一平台上做的笔记也会同步到其他平台。在 Zotero 6.0 正式版和移动端推出后我就是转向这种方式的。

采用另一种方式同步的同学，前面添加文献条目和附件的步骤都相同，你甚至也能在 iPad 上阅读（比如安装 OneDrive iPad 版），但是无法直接在 Zotero 移动端中像上面一样点击阅读，因为你是以链接形式添加的附件。

<!-- **Sync**

登录账号并取消勾选 Sync automatically（弃用）
附件通过坚果云与 iPad 同步（新增） -->

<!-- **Advanced**

Base Directory 设置为 OneDrive PDF 文件夹（弃用） -->

<!-- **ZotFile 插件设置**

Source Folder for Attaching Files 设置为下载文件夹
Custom Location 设置为上述 Base Directory（弃用）
Other Advanced Settings 选择 Always rename -->
