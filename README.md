# 在 Axios 中设置代理

[![推广](https://github.com/bright-cn/Rotating-Residential-Proxies/blob/main/50%25%20off%20promo%20%281%29.png)](https://www.bright.cn/proxy-types/residential-proxies) 

本 Axios 代理指南涵盖以下主题：

1. [Axios 与代理](#axios-与代理)
2. [在 Axios 中使用代理](#在-axios-中使用代理)
   - [HTTP/HTTPS 代理](#httphttps-代理)
   - [SOCKS 代理](#socks-代理)
3. [Axios 代理：高级用例](#axios-代理高级用例)
   - [全局设置代理](#全局设置代理)
   - [在 Axios 中处理代理身份验证](#在-axios-中处理代理身份验证)
   - [通过环境变量设置代理](#通过环境变量设置代理)
   - [实现动态代理](#实现动态代理)
4. [总结](#总结)

## Axios 与代理

[Axios](https://axios-http.com/) 是 JavaScript 生态系统中使用最广泛的 HTTP 客户端之一。它提供基于 Promise、易于使用且直观的 API，可用于执行 HTTP 请求以及处理自定义请求头、配置和 Cookie。

通过代理路由 Axios 请求，你可以隐藏自己的 IP 地址，让目标服务器更难识别并阻止你。

## 在 Axios 中使用代理

下面在 Axios 中设置 HTTP、HTTPS 或 SOCKS 代理。首先安装 `axios` npm 软件包：

```bash
npm install axios
```

在 Node.js 中，Axios 通过 [`proxy`](https://github.com/axios/axios#request-config) 配置原生支持 HTTP 和 HTTPS 代理。因此，如果你想在 Node.js 应用程序中使用 Axios 配合 HTTP/HTTPS 代理，无需进行其他操作。

如果你想使用非 HTTP/S 代理，则需要依赖 [Proxy Agents](https://github.com/TooTallNate/proxy-agents) 项目。该项目提供 `http.Agent` 实现，以便将 Axios 与采用不同协议的代理集成：

- HTTP 和 HTTPS 代理：[`https-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/https-proxy-agent)
- SOCKS、SOCKS5 和 SOCKS4：[`socks-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/socks-proxy-agent)
- PAC-\*：[`pac-proxy-agent`](https://github.com/TooTallNate/proxy-agents/blob/main/packages/pac-proxy-agent)

### HTTP/HTTPS 代理

HTTP/HTTPS 代理的 URL 应如下所示：

```
"<PROXY_PROTOCOL>://<PROXY_HOST>:<PROXY_PORT>"
```

- 对于 HTTP 代理，`<PROXY_PROTOCOL>` 为“http”；对于 HTTPS 代理，则为“https”。
- `<PROXY_HOST>` 通常是原始 IP 地址。
- `<PROXY_PORT>` 是代理服务器监听的端口。

例如，假设你的 HTTP 代理 URL 如下：

```
"http://47.88.62.42:80"
```

你可以按以下方式在 Axios 中设置此代理：

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "47.88.62.42",

        port: 80

    }

})
```

要验证上述 Axios 代理方法是否有效，请获取一个免费 HTTP 或 HTTPS 代理服务器的 URL。请尝试以下示例：

```
Protocol: HTTP; IP Address: 52.117.157.155; Port: 8002
```

完整的代理 URL 为 `http://52.117.157.155:8002`。

要验证代理是否正常工作，请请求 HTTPBin 项目的 [/ip](https://httpbin.io/ip) 端点。此公共 API 会返回传入请求的 IP，因此它应该返回代理服务器的 IP。

Node.js 脚本代码片段如下：

```js
import axios from "axios"

async function testProxy() {

    // perform the desired request through the HTTP proxy

const response = await axios.get("https://httpbin.io/ip", {
    proxy: {  
        protocol: "http",  
        host: "52.117.157.155",
        port: 8002
    }
});

    // print the result

    console.log(response.data)

}

testProxy()
```

执行该脚本，它应该输出：

```js
{ "origin": "52.117.157.155" }
```

> **警告**：\
> 运行该脚本时，你不会得到相同的结果，因为免费代理服务通常不可靠、速度慢、容易出错、会大量收集数据，而且存续时间很短。

### SOCKS 代理

如果尝试在代理配置对象的协议字段中设置“socks”字符串，你会收到以下错误：

```js
AssertionError [ERR_ASSERTION]: protocol mismatch

  // ...

 {

  generatedMessage: false,

  code: 'ERR_ASSERTION',

  actual: 'dada:',

  expected: 'http:',

  operator: '=='

}
```

这是因为 Axios 并不原生支持 SOCKS 代理。请将 `socks-proxy-agent` npm 库添加到项目依赖项中：

```bash
npm install socks-proxy-agent
```

使用此软件包，你可以在 Axios 中发出 HTTP 或 HTTPS 请求时连接到 SOCKS 代理服务器。

然后，从该库导入 SOCKS 代理 Agent 实现：

```js
const SocksProxyAgent = require("socks-proxy-agent")
```

如果你使用 ESM，则可以这样写：

```js
import { SocksProxyAgent } from "socks-proxy-agent"
```

假设你的 SOCKS 代理 URL 如下：

```
"socks://183.88.74.73:4153"
```

> **注意**：\
> 代理协议可以是“socks”“socks5”或“socks4”。

将其存储在变量中，并传递给 `SocksProxyAgent` 构造函数：

```js
const proxyURL = "socks://183.88.74.73:4153"

const proxyAgent = new SocksProxyAgent(proxyURL)
```

`SocksProxyAgent()` 会初始化一个 `http.Agent` 实例，以通过该代理 URL 执行 HTTP/HTTPS 请求。

现在，你可以按以下方式在 Axios 中使用 SOCKS 代理：

```js
axios.get(targetURL, { 

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

`httpAgent` 和 `httpsAgent` 分别定义执行 HTTP 和 HTTPS 请求时使用的自定义 Agent。换句话说，Axios 发出的 HTTP 或 HTTPS 请求将通过指定的 SOCKS 代理。类似地，你也可以使用 [`https-proxy-agent`](https://www.npmjs.com/package/https-proxy-agent) npm 软件包，以另一种方式在 Axios 中设置 HTTP/HTTPS 代理。

将所有代码整合起来：

```js
import axios from "axios"

import { SocksProxyAgent } from "socks-proxy-agent"

async function testProxy() {

    // replace with the URL of your SOCKS proxy 

    const proxyURL = "socks://183.88.74.73:4153"

    // define the HTTP/HTTPS proxy agent

    const proxyAgent = new SocksProxyAgent(proxyURL)

    // perform the request via the SOCKS proxy

    const response = await axios.get("https://httpbin.io/ip", { 

        httpAgent: proxyAgent,     

        httpsAgent: proxyAgent 

    })

    // print the result

    console.log(response.data) // { "origin": "183.88.74.73" }

}

testProxy()
```

点击链接，查看[如何在 Axios 中配置 SOCKS 代理](https://writech.run/blog/how-to-use-a-socks-proxy-in-axios-6c0355a2e013/)的其他示例。

## Axios 代理：高级用例

[![推广](https://github.com/bright-cn/LinkedIn-Scraper/blob/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/proxy-types/residential-proxies) 

### 全局设置代理

你可以直接在 Axios 实例中指定代理，以进行全局设置：

```js
const axiosInstance = axios.create({

    proxy: { 

        protocol: "<PROXY_PROTOCOL>", 

        host: "<PROXY_HOST>",

        port: "<PROXY_PORT>" 

    },

    // other configs...

})
```

如果你使用 Proxy Agents，也可以这样设置：

```js
// proxy Agent definition ...

const axiosInstance = axios.create({

    httpAgent: proxyAgent,     

    httpsAgent: proxyAgent 

})
```

以下是为 Axios 配置全局 SOCKS 代理的方法：

```js
import { SocksProxyAgent } from "socks-proxy-agent";

const proxyURL = "socks://183.88.74.73:4153";

// Create a SOCKS proxy agent
const proxyAgent = new SocksProxyAgent(proxyURL);

// Create an Axios instance with the SOCKS proxy
const axiosInstance = axios.create({
    httpAgent: proxyAgent, // for HTTP requests
    httpsAgent: proxyAgent, // for HTTPS requests
    // other configs...
});
```

现在，使用 `axiosInstance` 发出的所有请求都会自动通过指定的代理。

### 在 Axios 中处理代理身份验证

为了只允许付费用户访问优质代理，代理提供商会使用身份验证进行保护。如果在没有用户名和密码的情况下尝试连接需要身份验证的代理，将出现 [407 Proxy Authentication Required](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/407) 错误。

具体而言，需要身份验证的代理 URL 语法如下：

```
[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]
```

例如，用于连接需要身份验证的代理的真实 URL 可能如下：

```
http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391
```

在这种情况下，代理 URL 字段为：

- `<PROTOCOL>:HTTP`
- `<HOST>:156.127.0.192`
- `<PORT>:8391`
- `<USERNAME>:admin`
- `<PASSWORD>:lK4w90MEe45YIkOpk`

要在 Axios 中处理代理身份验证，请在 `proxy` 的 `authfield` 中指定用户名和密码：

```js
axios.get(targetURL, {

    proxy: { 

        protocol: "http", 

        host: "156.127.0.192",

        port: "8381",

        auth: {

            username: "admin",

            password: "lK4w90MEe45YIkOpk"

        }

    }

})
```

如果你使用 Proxy Agents，则可以通过两种方式处理身份验证：

1. 直接在代理 URL 中添加凭据：

```js
var proxyAgent = new SocksProxyAgent("http://admin:lK4w90MEe45YIkOpk@156.127.0.192:8391")
```

2. 在 [URL](https://nodejs.org/api/url.html) 对象中设置 `username` 和 `password` 选项：

```js
const proxyOpts = new URL("http://156.127.0.192:8391")

proxyOpts.username = "admin"

proxyOpts.password = "lK4w90MEe45YIkOpk"

const proxyAgent = new SocksProxyAgent(proxyOpts)
```

相同的方法也适用于 HttpsProxyAgent。

### 通过环境变量设置代理

在 Axios 中全局配置代理的另一种方法是设置以下环境变量：

- `HTTP_PROXY`：用于 HTTP 请求的代理服务器 URL。
- `HTTPS_PROXY`：用于 HTTPS 请求的代理服务器 URL。

在 Linux 或 macOS 上，你可以按如下方式设置：

```bash
export HTTP_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"

export HTTPS_PROXY = "[<PROTOCOL>://]<USERNAME>:<PASSWORD>@<HOST>[:<PORT>]"
```

Axios 检测到这些环境变量后，会从中读取代理设置，包括用于身份验证的凭据。将 `proxy` 字段设为 `false`，可让 Axios 忽略这些环境变量。请注意，你还可以定义 `NO_PROXY` 环境变量，以逗号分隔的形式列出不应使用代理的域名。

请注意，[在 cURL 中使用代理](https://www.bright.cn/blog/proxy-101/curl-with-proxies)时，同样的机制也适用。

### 实现动态代理

为了防止目标网站封禁代理的 IP 地址，请确保每个请求都来自不同的代理服务器：

1. 定义一个对象列表，其中每个对象都包含连接到不同代理所需的信息。
2. 在每次请求前随机选择一个代理对象。
3. 在 Axios 中配置所选代理。

上述方法假设你拥有一组可靠的代理服务器，例如 Bright Data 提供的[动态代理](https://www.bright.cn/solutions/rotating-proxies)。

## 总结

Bright Data 拥有世界领先的代理服务器，为《财富》500 强企业及超过 20,000 家客户提供服务。其全球代理网络包括：

*   [数据中心代理](https://www.bright.cn/proxy-types/datacenter-proxies)：超过 770,000 个数据中心 IP。
*   [住宅代理](https://www.bright.cn/proxy-types/residential-proxies)：覆盖 195 多个国家/地区，拥有超过 7200 万个住宅 IP。
*   [ISP 代理](https://www.bright.cn/proxy-types/isp-proxies)：超过 700,000 个 ISP IP。

立即[创建免费的 Bright Data 账户](https://www.bright.cn/#popup-127701)，试用我们的代理服务器。
