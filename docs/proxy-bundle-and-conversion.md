# 机场订阅聚合与转换

主要用到的工具是[阿里云函数计算]和 [subconverter]，前者用于机场订阅聚合，后者用于转换订阅。参考这篇[博客]和 subconverter 文档写成。

## 订阅聚合

进入函数计算后台，先切换到中国香港集群。在“服务及函数”中创建服务，填写名称，如 proxy，其他保持默认即可。

在刚创建的服务中新建函数，选择“使用内置运行时创建”，函数名称设置为 subscribe。

确保选中“处理 HTTP 请求”，然后在下方“高级配置”中将“实例并发度”设置为 100，“执行超时时间”设置为推荐的 600。

在“环境变量”中添加 `URL` 变量，值为订阅链接，有多个链接的话用`|`分隔。再添加一个 `TOKEN` 变量，值的格式为 `<用户名>=<token>`，同样以 `|` 分隔。上面博客中有相关设置截图。

完成后，点击“创建”会进入代码编辑页面，用的是 VS Code Server。调出终端，执行 `npm install axios`。修改 `index.js` 为以下代码，记得点击部署函数。

```js
const axios = require('axios');
axios.defaults.timeout = 3000;

const URLS = (process.env.URL || '')
  .split('|')
  .filter((item) => item.length > 0);
const TOKENS = Object.assign(
  {},
  ...(process.env.TOKEN || '')
    .split('|')
    .filter((item) => item.length > 0)
    .filter((item) => item.indexOf('=') !== -1)
    .map((item) => item.split('='))
    .map((item) => ({ [item[1]]: item[0] }))
);

const TESTSUBSCRIBE='dm1lc3M6Ly9ldzBLSUNBaWRpSTZJQ0l5SWl3TkNpQWdJbkJ6SWpvZ0l1YTFpK2l2bFNJc0RRb2dJQ0poWkdRaU9pQWlNVEkzTGpBdU1DNHhJaXdOQ2lBZ0luQnZjblFpT2lBaU1USXpORFVpTEEwS0lDQWlhV1FpT2lBaU16QXdaRGN6T1RZdE1tUXlPQzAwWmpKaUxUaG1PV1l0TXpjMU5UQTVZbVZpTVROaElpd05DaUFnSW1GcFpDSTZJQ0l3SWl3TkNpQWdJbk5qZVNJNklDSmhkWFJ2SWl3TkNpQWdJbTVsZENJNklDSjBZM0FpTEEwS0lDQWlkSGx3WlNJNklDSnViMjVsSWl3TkNpQWdJbWh2YzNRaU9pQWlJaXdOQ2lBZ0luQmhkR2dpT2lBaUlpd05DaUFnSW5Sc2N5STZJQ0lpTEEwS0lDQWljMjVwSWpvZ0lpSU5DbjA9DQo='

const fromBase64 = (s) => Buffer.from(s, 'base64').toString();
const toBase64 = (s) => Buffer.from(s).toString('base64');

const removePrefix = (s, p) => s.startsWith(p) ? s.slice(p.length) : s;

const get = (url, resolve, reject) =>
  axios
    .get(url)
    .then((resp) => resp.data)
    .then(resolve)
    .catch(reject);

async function getData() {
  try {
    return toBase64(
      (
        await Promise.all(
          URLS.map(
            (url) => new Promise((resolve, reject) => get(url, resolve, reject))
          )
        )
      )
        .map((encoded) =>
          fromBase64(encoded)
            .split('\n')
            .filter((line) => line.length > 0)
        )
        .reduce((pre, cur) => pre.concat(cur))
        .join('\n')
    );
  } catch (e) {
    return e.toString();
  }
}

exports.handler = async (req, resp, context) => {
  console.log(req.path);
  if (removePrefix(req.path, "/subscribe") === '/test') {
    console.log(req.clientIP);
    resp.send(TESTSUBSCRIBE);
    return;
  }

  const token = req.queries['token'];
  console.log(URLS, TOKENS);
  console.log(req.clientIP, token, TOKENS[token]);
  const res = !!TOKENS[token] ? await getData() : '';
  resp.send(res);
};
```

## 自定义域名

回到函数计算主管理界面，在“域名管理”中添加自定义域名。设置好域名解析后，修改路由配置为 `/subscribe`，其他栏下拉进行相应选择。完成后，再照此添加一条路由 `/subscribe/*`。你也可以为域名申请并启用证书，最后点击创建即可。

## 搭建 subconverter

那篇博客中 subconverter 部分我试过，经常报错，也懒得去查阿里云的文档。干脆，直接用 [Docker][subconverter-docker] 搭建算了。你可以通过 [Composerize] 网站，将 Docker 命令转换成 Docker Compose 更方便些。

## 使用方式

根据[相关文档][中文文档]，调用地址中的 `url` 参数和远程引用的 `config` 文件链接，都必须经过 [URL Encode] 处理。以下面为例进行简单说明。

`https://<your_subconverter_server>/sub?target=clash&new_name=true&url=<your_subscription_link>&filename=<filename>&interval=21600&insert=false&config=https%3A%2F%2Fraw.githubusercontent.com%2FFDUZS%2Fsubconverter-config%2Fmain%2Fconfig.ini`

- `<your_subconverter_server>` 为上面搭建的 subconverter 后端，如果用 IP 访问或无证书，则为 `http`
- `url` 为上面订阅聚合链接，记得带 token 参数并 [URL Encode] 处理
- `filename` 是在 [Clash for Windows] 等软件中导入后显示的名字
- `interval` 是在 [Clash for Windows] 等软件中导入后自动更新周期，单位为秒
- `config` 即外部配置文件，需经 [URL Encode] 处理，可直接使用上述例子中的
- 其他更多参数可查询相关文档

[阿里云函数计算]: https://fcnext.console.aliyun.com/cn-hongkong/services
[subconverter]: https://github.com/tindy2013/subconverter
[博客]: https://www.ohyee.cc/post/note_lambda_v2ray_clash
[subconverter-docker]: https://github.com/tindy2013/subconverter/blob/master/README-docker.md
[Composerize]: https://www.composerize.com/
[中文文档]: https://github.com/tindy2013/subconverter/blob/master/README-cn.md
[URL Encode]: https://www.urlencoder.org/
[Clash for Windows]: https://docs.cfw.lbyczf.com/
