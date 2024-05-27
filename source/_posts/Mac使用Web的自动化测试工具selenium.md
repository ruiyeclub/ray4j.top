---
title: Mac使用Web的自动化测试工具selenium
date: 2024-05-27 17:03:33
tags:
---

> 使用 [selenium](https://baike.baidu.com/item/Selenium/18266) 模拟浏览器渲染数据，需要依赖各浏览器的驱动才能完成，因此需要单独安装chrome driver。

### 一、下载对应`chrome`浏览器驱动

1. 下载地址：[chrome driver](https://googlechromelabs.github.io/chrome-for-testing/)

![image-20240527155408011](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20240527155408011.png)

2. 安装 chrome driver

3. 解压完 zip 压缩包之后，拷贝二进制文件到 /usr/local/bin 目录下即可

   ```shell
   # 拷贝
   cp chromedriver-mac-arm64/chromedriver /usr/local/bin
    
   # 授权
   cd /usr/local/bin
   xattr -d com.apple.quarantine chromedriver
   ```

### 二、python调用selenium

1. 安装selenium

```python
pip install selenium
```

2. 唤起谷歌浏览器

```python
from selenium import webdriver
# 启动谷歌浏览器
driver=webdriver.Chrome() 
# 访问一个网页
driver.get("http://www.zhihu.com") 
# 退出浏览器
driver.quit() 
```

3. 调用页面接口

```python
from selenium import webdriver

# 创建一个ChromeOptions对象，用于设置Chrome浏览器的启动选项。
options = webdriver.ChromeOptions()
# 使Chrome以无头模式运行（即不显示浏览器窗口）。这通常用于在服务器环境中进行自动化操作。
options.add_argument('--headless')
# 禁用GPU硬件加速。这是为了在某些无头模式的情况下避免一些bug。
options.add_argument('--disable-gpu')
# 禁用沙盒模式。沙盒模式通常用于安全性，但在某些环境中可能会导致问题，因此这里将其禁用。
options.add_argument('--no-sandbox')
# 设置浏览器的user-agent字符串。这使得请求看起来像是来自一个正常的桌面浏览器，而不是一个自动化工具，从而减少被反爬虫机制识别的风险。
options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36')

driver = webdriver.Chrome(options=options)
# 让浏览器访问指定的URL，即https://investing.com。
driver.get('https://investing.com')
# 使用浏览器执行JavaScript代码，通过fetch API发送一个HTTP请求到指定的API endpoint，并返回其响应。该响应将被解析为JSON格式并存储在response变量中。
response = driver.execute_script("return fetch('https://api.investing.com/api/financialdata/7310/historical/chart/?interval=P1D&pointscount=160').then(res => res.json())")
print(response)

driver.quit()
```

