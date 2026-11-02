# 绕过 Cloudflare：最佳实践

[![促销](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/) 

本指南介绍如何绕过 Cloudflare 的安全机制，成功抓取网站而不被封禁。

- [使用代理解决方案](#使用代理解决方案)
- [伪装 HTTP 标头](#伪装-http-标头)
- [接入 CAPTCHA 破解服务](#接入-captcha-破解服务)
- [使用强化型无头浏览器](#使用强化型无头浏览器)
- [使用 Cloudflare 破解工具](#使用-cloudflare-破解工具)
- [高级技术](#高级技术)
- [集成 Bright Data 解决方案](#集成-bright-data-解决方案)

## 了解 Cloudflare 的机制

Cloudflare 的[网页应用防火墙](https://www.cloudflare.com/application-services/products/waf/) (WAF) 可保护 Web 应用免受 DDoS 和零日攻击。它在 Cloudflare 的全球网络上运行，能够实时阻止攻击，并使用专有算法根据以下多种特征识别和拦截恶意爬虫：

- [**TLS 指纹**](https://www.bright.cn/blog/web-data/tls-fingerprinting)：JA3 指纹可识别客户端及其功能和配置，并验证客户端是否真实。
- **[HTTP/2 指纹](https://www.blackhat.com/docs/eu-17/materials/eu-17-Shuster-Passive-Fingerprinting-Of-HTTP2-Clients-wp.pdf)**：使用 HTTP/2 参数与已知的爬虫签名进行匹配。
- **HTTP 详细信息**：检查标头和 Cookie 中是否存在类似爬虫的配置。
- **JavaScript 指纹**：收集浏览器、操作系统和硬件详细信息，以区分爬虫。
- **行为分析**：通过机器学习监控请求频率、鼠标移动和空闲时间，以检测爬虫。

如果检测到类似爬虫的活动，Cloudflare 会在后台发起 JavaScript 质询；若未能通过，则会显示 CAPTCHA。

## 绕过 Cloudflare 的技术

Cloudflare 的专有爬虫检测并非万无一失，因此你需要进行尝试，找到最适合自身需求的方法。

### 使用代理解决方案

Cloudflare 会将来自单个 IP 的过多请求标记为爬虫活动。要避免这种情况，请使用[优质住宅代理](https://www.bright.cn/proxy-types/residential-proxies)。不过，如果启用了 User-Agent 检查，你还需要伪装 User-Agent。

### 伪装 HTTP 标头

[HTTP 标头](https://www.bright.cn/blog/web-data/http-headers-for-web-scraping)会暴露客户端的详细信息。Cloudflare 会检查这些信息，以区分发送大量标头的真实浏览器与仅发送少量标头的抓取工具。大多数爬虫工具都允许你修改标头，以模拟真实浏览器。常见标头包括：

#### User-Agent 标头

`User-Agent` 标头会暴露浏览器和操作系统。Cloudflare 可能会拦截类似爬虫的 User-Agent，因此将其伪装成真实浏览器（例如 Chrome、Firefox 或 Safari）有助于避免被检测。例如，在 Python 的 [`requests` 库](https://pypi.org/project/requests/)中，你可以这样设置：

```python
import requests
 
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

#### Referrer 标头

Cloudflare 会检查 `Referer` 标头以验证请求来源。使用有效 URL 对其进行伪装，可以让请求看起来可信。

```python
import requests
 
headers = {
    'Referer': 'https://trusted-website.com'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

#### Accept 标头

`Accept` 标头用于指定客户端可以处理哪些内容类型。模拟真实浏览器中详细的 `Accept` 标头有助于避免被检测：

```python
import requests
 
headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8'
}
 
response = requests.get('http://httpbin.org/headers', headers=headers)
 
print(response.status_code)
print(response.text)
```

Cloudflare 还会检查标头是否不匹配或已过时。例如，将 Firefox User-Agent 与 `Sec-CH-UA-Full-Version-List` 搭配使用可能会导致请求被拦截，因为 Firefox 不支持它。

### 接入 CAPTCHA 破解服务

当其他检测方法无法得出明确结果时，Cloudflare 可能会向可疑客户端显示 CAPTCHA。其 Turnstile 系统会运行轻量级的非交互式质询，但也可以切换为对爬虫工具发起挑战的交互式 CAPTCHA。此外，许多服务还会使用人工破解方式绕过这些 CAPTCHA。阅读我们关于[最佳验证码破解工具]( https://www.bright.cn/blog/web-data/best-captcha-solvers)的文章，为你的使用场景找到理想的服务。 

### 使用强化型无头浏览器

要绕过 Cloudflare 的 JavaScript 质询，你的爬虫必须模拟真实浏览器行为，包括执行 JavaScript、处理 Cookie，以及模拟滚动、鼠标移动和点击等操作。[Selenium](https://www.selenium.dev/) 等工具可以实现这些功能，但[无头浏览器](https://www.bright.cn/blog/web-data/best-headless-browsers)往往会暴露自身特征（例如通过 `navigator.webdriver`）。[undetected_chromedriver](https://github.com/ultrafunkamsterdam/undetected-chromedriver) 和 [puppeteer-extra-plugin-stealth](https://www.bright.cn/blog/how-tos/avoid-getting-blocked-with-puppeteer-stealth) 等插件有助于掩盖这些特征。

以下是使用 undetected_chromedriver 的示例：

```python
import undetected_chromedriver.v2 as uc
driver = uc.Chrome()
with driver:
    driver.get('https://example.com')
```

你可以将无头浏览器与[高质量代理服务](https://www.bright.cn/proxy-types)搭配使用，使其更能抵御 Cloudflare 的检测：

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

浏览器经常更新，这可能会暴露无头浏览器的特征，而 Cloudflare 不断演进的算法可以利用这些特征。因此，必须定期维护插件，否则它们可能会失效。

### 使用 Cloudflare 破解工具

专用的 [Cloudflare 破解服务](https://github.com/bright-cn/cloudflare-captcha-solver)可以暂时绕过基础保护。例如，cloudscraper 使用 JavaScript 引擎模拟浏览器支持，但由于其更新不及时，实际效果可能有所下降。

### 高级技术

Cloudflare 使用多种爬虫检测方法，因此单一技术并不足以绕过检测。你需要组合多种方法来模拟真实用户。例如，使用强化型无头浏览器、模拟人类鼠标移动（如 [B 样条曲线](https://stackoverflow.com/a/48690652)）、轮换住宅代理以避免 IP 被封禁，并使用 [Hazetunnel](https://github.com/daijro/hazetunnel) 等工具复现真实浏览器指纹。再加入 CAPTCHA 破解工具，可以进一步提高绕过 Cloudflare 防御机制的成功率。

## 集成 Bright Data 解决方案

[Bright Data 的网络解锁器](https://github.com/bright-cn/web-unlocker-api)利用 AI 应对反爬虫措施（例如浏览器指纹识别、验证码破解、IP 轮换和请求重试），成功率高达 99.99%，从而简化绕过 Cloudflare 爬虫检测的过程。它会自动选择最佳代理并提供简单的身份验证凭据，让你能够像使用任何标准代理服务器一样使用它。你可以像使用其他代理服务器一样使用它：

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

[Bright Data 的抓取浏览器](https://github.com/luminati-io/scraping-browser)通过在远程浏览器上运行你的代码来绕过 Cloudflare，该浏览器会使用多个代理解锁网站。它可与 [Puppeteer](https://www.bright.cn/products/scraping-browser/puppeteer)、[Selenium](https://www.bright.cn/products/scraping-browser/selenium) 和 [Playwright](https://www.bright.cn/products/scraping-browser/playwright) 集成，提供完整的无头浏览器体验。

## 总结

规避 Cloudflare 可能很复杂，而且成功率也各不相同。与其勉强拼凑解决方案，不如考虑使用 [Bright Data 的产品](https://www.bright.cn/products)，例如网络解锁器、抓取浏览器和网页爬虫工具 API。只需几行代码，你就能获得更高的成功率，而无需费心管理复杂的解决方案。
