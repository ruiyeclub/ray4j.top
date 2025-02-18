---
title: 绕过cloudflare 5秒盾的技术方案解析
date: 2025-02-18 15:03:59
tags:
---

> Cloudflare 的 5秒盾（Under Attack Mode）是网站防护的常见手段，通过 JavaScript 挑战验证访客真实性。很多网站会使用Cloudflare提供安全服务，导致爬虫的时候显示403状态。


![55187d199c2a9079caff09622c6f78f9933ed773](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/55187d199c2a9079caff09622c6f78f9933ed773.png)

## 一、解决方案及代码示例

#### 1. 使用 cloudscraper 库

**原理**：模拟 Cloudflare 预期的请求特征

```python
import cloudscraper

scraper = cloudscraper.create_scraper()
response = scraper.get("https://target-site.com")
print(response.text)
```

#### 2. 使用 curl_cffi 库

**优势**：支持多浏览器指纹模拟

```python
from curl_cffi import requests

response = requests.get(
    "https://target-site.com",
    impersonate="chrome110"  # 支持 chrome99-110 / edge99-110
)
print(response.text)
```

#### 3. 第三方 API 服务

**推荐服务**：

- ScrapingBee（自带代理轮转）
- ScraperAPI
- ZenRows

```python
import requests

API_KEY = "your_api_key"
response = requests.get(
    f"https://api.scrapingbee.com/v1/?api_key={API_KEY}&url=target-site.com"
)
```

#### 4. FlareSolverr 服务

**架构**：基于 Puppeteer 的中间件服务

部署步骤：

```bash
docker run -d \
  --name flaresolverr \
  -p 8191:8191 \
  -e LOG_LEVEL=info \
  --restart unless-stopped \
  ghcr.io/flaresolverr/flaresolverr:latest
```

调用示例：

```python
import requests

payload = {
    "cmd": "request.get",
    "url": "https://target-site.com",
    "maxTimeout": 60000
}
response = requests.post("http://localhost:8191/v1", json=payload)
```

## 二、方案对比分析

| 方案         | 执行速度 | 维护成本 | 隐匿性 | 适用场景         |
| ------------ | -------- | -------- | ------ | ---------------- |
| cloudscraper | ★★★★     | 低       | 中     | 中小规模常规采集 |
| curl_cffi    | ★★★★★    | 低       | 高     | 需要最新指纹场景 |
| 第三方API    | ★★★      | 中       | 高     | 企业级高频采集   |
| FlareSolverr | ★★       | 高       | 极高   | 复杂JS验证场景   |

## 三、对抗升级建议

1. **IP轮换策略**：结合住宅代理使用（推荐 BrightData）
2. **指纹随机化**：定期更换 User-Agent/TLS 指纹
3. **请求限速**：设置 3-10 秒随机延迟
4. **头信息校验**：携带完整 Accept-Encoding/Cookie 等头