# 绕过 Cloudflare：最佳实践

[![推广](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/) 

本指南将介绍如何绕过 Cloudflare 的安全机制，并成功抓取不会被阻止的网站。

- [使用代理解决方案](#使用代理解决方案)
- [伪造 HTTP 头](#伪造-http-头)
- [实施 CAPTCHA 解决服务](#实施-captcha-解决服务)
- [使用强化的无头浏览器](#使用强化的无头浏览器)
- [使用 Cloudflare 解题工具](#使用-cloudflare-解题工具)
- [高级技巧](#高级技巧)
- [整合 Bright Data 解决方案](#整合-bright-data-解决方案)

## 了解 Cloudflare 的机制

Cloudflare 的[Web应用防火墙（WAF）](https://www.cloudflare.com/application-services/products/waf/)通过其全球网络来保护 Web 应用免受 DDoS 和零日攻击。它能实时阻止攻击，并利用专有算法，根据多种特征来识别并阻止恶意机器人，包括：

- [**TLS 指纹**](https://www.bright.cn/blog/web-data/tls-fingerprinting)：JA3 指纹用来识别客户端及其功能、配置，以验证客户端是否是真实用户。
- **[HTTP/2 指纹](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shuster-Passive-Fingerprinting-Of-HTTP2-Clients-wp.pdf)**：利用 HTTP/2 参数与已知的机器人特征进行匹配。
- **HTTP 细节**：检查头部信息和 Cookie 以识别疑似机器人的配置。
- **JavaScript 指纹**：收集浏览器、操作系统和硬件信息，以区分机器人与真实用户。
- **行为分析**：通过机器学习监测请求频率、鼠标移动、空闲时间等来检测机器人。

一旦 Cloudflare 检测到可疑的机器人活动，就会发起背景 JavaScript 挑战；若无法通过则会要求输入 CAPTCHA。

## 绕过 Cloudflare 的技巧

Cloudflare 的专有机器人检测并非牢不可破，具体解决方案需要结合自身需求来不断试验和优化。

### 使用代理解决方案

Cloudflare 会根据同一 IP 发送过多请求来识别并阻止机器人。为避免这一点，可以使用[优质住宅代理](https://www.bright.cn/proxy-types/residential-proxies)。但如果对方还会检测 User-Agent，则需要进行相应的伪造。

### 伪造 HTTP 头

[HTTP 头](https://www.bright.cn/blog/web-data/http-headers-for-web-scraping)可以暴露客户端的详细信息。Cloudflare 会利用它们来区分真实浏览器和只发送少数头部信息的爬虫。大多数爬虫工具都允许你修改头部来模拟真实浏览器。常见的头部包括：

#### User-Agent 头

`User-Agent` 头会暴露所用浏览器与操作系统。Cloudflare 可能会阻止明显像机器人的 User-Agent，因此可将其伪装成常用浏览器（如 Chrome、Firefox、Safari）来提高成功率。下面是使用 Python [`requests` 库](https://pypi.org/project/requests/)进行设置的示例：

```python
import requests
 
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

#### Referer 头

Cloudflare 会检查 `Referer` 头来验证请求的来源。将其伪造为一个可信的 URL，能使请求看起来更可信。

```python
import requests
 
headers = {
    'Referer': 'https://trusted-website.com'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

#### Accept 头

`Accept` 头用于声明客户端可处理的内容类型。模拟真实浏览器中详细的 `Accept` 头，可帮助避免被识别为机器人：

```python
import requests
 
headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

Cloudflare 也会检查头部的不匹配或是否过时。例如，如果你使用的是 Firefox 的 User-Agent，但包含 `Sec-CH-UA-Full-Version-List` 头，可能会被阻断，因为 Firefox 并不支持该头部。

### 实施 CAPTCHA 解决服务

当其他检测手段都无法确定客户端是否可信时，Cloudflare 可能会向可疑客户端展示 CAPTCHA。它的 Turnstile 系统通常提供轻量级的无交互挑战，但在必要时也会切换为需人工输入的交互式 CAPTCHA。很多服务使用真人或其他方式来解决这些验证码。可参阅我们的[最佳 CAPTCHA 解决方案](https://www.bright.cn/blog/web-data/best-captcha-solvers)文章，筛选最适合自己的服务。 

### 使用强化的无头浏览器

如果想绕过 Cloudflare 的 JavaScript 挑战，你的爬虫必须模拟真实浏览器：执行 JavaScript、处理 Cookie、模拟用户滚动、鼠标移动和点击等操作。[Selenium](https://www.selenium.dev/) 等工具能完成这些，但许多[无头浏览器](https://www.bright.cn/blog/web-data/best-headless-browsers)本身也会暴露特征（例如 `navigator.webdriver`）。你可以使用 [undetected_chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver)、[puppeteer-extra-plugin-stealth](https://www.bright.cn/blog/how-tos/avoid-getting-blocked-with-puppeteer-stealth) 等插件来隐藏这些特征。

以下是使用 undetected_chromedriver 的示例：

```python
import undetected_chromedriver.v2 as uc
driver = uc.Chrome()
with driver:
    driver.get('https://example.com')
```

你也可以将无头浏览器与[高质量代理服务](https://www.bright.cn/proxy-types)结合使用，以加强对 Cloudflare 的规避能力：

```python
chrome_options = uc.ChromeOptions()

proxy_options = {
    'proxy': {
        'http': 'HTTP_PROXY_URL',
        'https': 'HTTPS_PROXY_URL'
    }
}

driver = uc.Chrome(
    options=chrome_options,
    seleniumwire_options=proxy_options
)
```

浏览器频繁更新会带来新的无头检测特征，而 Cloudflare 的算法也在不断进化，可能利用这些新特征。因此，这些反侦察插件需持续维护，否则很容易失效。

### 使用 Cloudflare 解题工具

专门的 [Cloudflare 解题服务](https://github.com/bright-cn/cloudflare-captcha-solver)可以在短期内绕过一些基本防护。例如，cloudscraper 通过 JavaScript 引擎来模拟浏览器功能，但其更新往往滞后，因此效果会受影响。

### 高级技巧

Cloudflare 会结合多种识别手段，因此单一手段往往不足以完全绕过。建议综合多种方法来最大化模拟真实用户行为。例如，使用强化的无头浏览器、模拟真实的鼠标移动（如利用[B-spline 曲线](https://stackoverflow.com/a/48690652)）、轮换住宅代理以避免 IP 封禁，并使用像 [Hazetunnel](https://github.com/daijro/hazetunnel) 这样的工具来模仿真实浏览器指纹。再结合 CAPTCHA 解决方案，能显著提高绕过 Cloudflare 检测的成功率。

## 整合 Bright Data 解决方案

[Bright Data 的 Web Unlocker](https://github.com/bright-cn/web-unlocker-api) 借助 AI 技术来自动应对 Cloudflare 的反爬机制（涵盖浏览器指纹、CAPTCHA 解决、IP 轮换、请求重试等），成功率高达 99.99%。它会自动选择最佳代理，使用方式与标准代理服务器类似，只需简单的身份验证即可。使用方式示例如下：

```python
import requests

host = 'brd.superproxy.io'
port = 22225

username = 'brd-customer-<customer_id>-zone-<zone_name>'
password = '<zone_password>'

proxy_url = f'http://{username}:{password}@{host}:{port}'

proxies = {
    'http': proxy_url,
    'https': proxy_url
}

url = "http://lumtest.com/myip.json"
response = requests.get(url, proxies=proxies)
print(response.json())
```

[Bright Data 的 Scraping Browser](https://github.com/bright-cn/scraping-browser) 则直接在远程浏览器中运行你的代码，并结合多重代理来解锁站点。它可以与 [Puppeteer](https://www.bright.cn/products/scraping-browser/puppeteer)、[Selenium](https://www.bright.cn/products/scraping-browser/selenium) 和 [Playwright](https://www.bright.cn/products/scraping-browser/playwright) 集成，提供完整的无头浏览器环境。

## 结论

绕过 Cloudflare 可能较为复杂，且成功率会因策略不同而差异明显。与其尝试东拼西凑的方案，不如考虑使用 [Bright Data 的产品](https://www.bright.cn/products)，如 Web Unlocker、Scraping Browser 或 Web Scraper API。仅需少量代码即可实现更高的成功率，同时免去对复杂技术细节的管理和维护负担。
