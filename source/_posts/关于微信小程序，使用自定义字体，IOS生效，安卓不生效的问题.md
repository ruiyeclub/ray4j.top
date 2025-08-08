---
title: 关于微信小程序，使用自定义字体，IOS生效，安卓不生效的问题
date: 2025-08-08 15:23:49
tags:
---

> 微信小程序在 iOS 设备上引用字体正常，但在 安卓 设备上无法加载时，通常由 Nginx 配置问题导致。以下是具体解决方案：

### Nginx配置调整

1. ‌**添加跨域权限**‌
   在Nginx配置文件中，针对字体文件（如`.woff`、`.ttf`等）的请求头添加跨域权限设置：

   ```sh
   		# 字体文件跨域设置
       location ~* \.(ttf|woff|woff2)$ {
           add_header Access-Control-Allow-Origin *;
           expires      30d;
           error_log /dev/null;
           access_log /dev/null;
       }
   ```

2. ‌**设置响应头参数**‌
   确保响应包含以下头部信息：

    - `Access-Control-Allow-Origin: *`（允许所有来源跨域）
    - `Cache-Control: no-cache`（禁用缓存） ‌

3. ‌**清理小程序缓存**‌
   若配置完成后仍无法生效，需在微信开发者工具中清理小程序缓存后重新测试。 ‌

4. **完整配置**

```
server
{
    listen 80;
    listen 443 ssl http2 ;
    server_name img.**.ruiyeclub.cn;
    index index.html index.htm default.htm default.html;
    root /www/wwwroot/img.**.ruiyeclub.cn;

    #REWRITE-START URL重写规则引用,修改后将导致面板设置的伪静态规则失效
    include /www/server/panel/vhost/rewrite/html_img.**.ruiyeclub.cn.conf;
    #REWRITE-END

    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.env|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }

    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }

    #禁止在证书验证目录放入敏感文件
    if ( $uri ~ "^/\.well-known/.*\.(php|jsp|py|js|css|lua|ts|go|zip|tar\.gz|rar|7z|sql|bak)$" ) {
        return 403;
    }

    # 字体文件跨域设置
    location ~* \.(ttf|woff|woff2)$ {
        add_header Access-Control-Allow-Origin *;
        expires      30d;
        error_log /dev/null;
        access_log /dev/null;
    }

    location ~ .*\\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        add_header Cache-Control "public, max-age=31536000"; #my 设置静态资源的缓存时间为1年（单位：秒）

        expires      30d;
        error_log /dev/null;
        access_log /dev/null;
    }

    location ~ .*\\.(js|css)?$
    {
        expires      12h;
        error_log /dev/null;
        access_log /dev/null;
    }
    access_log  /www/wwwlogs/img.**.ruiyeclub.cn.log;
    error_log  /www/wwwlogs/img.**.ruiyeclub.cn.error.log;
}
```

### 注意事项

- 字体文件路径需与配置一致，例如存放在`/www/wwwroot/***/public/static`目录下。 ‌
- 不同品牌安卓手机对跨域策略的兼容性存在差异，部分机型需额外配置域名白名单或业务域名。