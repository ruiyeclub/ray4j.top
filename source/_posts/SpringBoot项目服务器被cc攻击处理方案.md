---
title: SpringBoot项目服务器被cc攻击处理方案
date: 2026-06-01 15:53:47
tags:
---


> **痛点直击**：攻击者不针对数据库，不针对服务器进程，而是专门针对 没有token校验接口发起海量请求。你的带宽先被打满，SpringBoot 连被访问的机会都没有。

如果你的 SpringBoot 项目正在遭受针对 **Login 登录接口** 的猛烈攻击，你会发现一个奇怪的现象：服务器 CPU 可能还没满，但 **带宽（出入流量）直接 100%**，导致正常用户无法访问，SSH 连不上，服务瘫痪。

这种攻击通常被称为 **CC 攻击（Challenge Collapsar）**，是 DDoS 的一种应用层变种。对于这种攻击，**千万别试图在 SpringBoot 代码里解决**，必须在 **Cloudflare (CF)** 的边缘节点上将其拦截。

以下是针对 Login 接口被攻击的 **“止血+加固”** 全流程。

------

## 一、 紧急止血：Cloudflare 上的“战争迷雾”

![image-20260601152334409](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601152334409.png)

既然攻击者是通过域名攻击，我们必须利用 Cloudflare 的全球网络，在流量到达你的源站服务器之前，就把它们吃掉。

### 1. 开启 “Under Attack Mode”（正在被攻击模式）

这是 CF 的大招，虽然会影响一点用户体验，但能保住服务器。

![image-20260601151121786](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601151121786.png)

开启 Cloudflare Under Attack Mode（“5秒盾”）的步骤如下：

1. 登录 [Cloudflare 仪表板](https://dash.cloudflare.com)，选择对应域名。  
2. 点击顶部菜单 “Security”（安全）选项卡。  
3. 在 “Security Level” 下方找到 “Under Attack Mode” 开关，将其设为 “On”（变绿即生效）。  
4. 开启后，访客访问网站会弹出 JavaScript 质询页面（俗称“5秒盾”），需等待或完成验证才能进入。

⚠️ 注意：  

- 此模式仅在遭遇 DDoS/CC 攻击时临时启用，日常开启会降低正常用户体验（增加加载延迟）。  
- 需确保域名已通过 Cloudflare 接入（DNS 已代理，状态为橙色云朵）。  
- 免费计划支持该功能，但无法自定义质询页面内容（需 Enterprise 计划）。  
- 若需 API 自动化开启，可发送 PATCH 请求至 `https://api.cloudflare.com/client/v4/zones/{zone_id}/settings/security_level`，设置 `value=under_attack`（需 Global API Key 和 Zone ID）。

关闭时请手动切回“High”或默认安全等级，避免影响正常流量。

### 2. 针对 `/login`接口配置 WAF 速率限制（最关键）

![image-20260601153234925](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601153234925.png)

通用的限流不够，我们要对 `login`接口进行“精准打击”。

**创建 WAF 自定义规则：**

![image-20260601153151841](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601153151841.png)

1. 点 **Create rate limiting rule**（创建速率限制规则）

2. Rule name（规则名）

   ```
   Block Login Brute Force
   ```

   （随便写，能识别就行）

3. When incoming requests match（匹配条件，精准锁定登录接口）

推荐用 **表达式模式（Edit expression）** 更精准Cloudflare：

```
http.request.uri.path eq "/api/auth/login" and http.request.method eq "POST"
```

- 把 `/api/auth/login` 换成你真实的登录接口路径
- 只拦截 POST（登录一般都是 POST）Cloudflare

4. With the same characteristics（按什么限流）

选：**IP（源 IP）**

含义：**同一个 IP** 才累计计数，完美防单 IP 暴力破解Cloudflare。

5. When rate exceeds（限流阈值，你要的 “10 秒 10 次”）

- **Requests（次数）**：`10`
- **Period（时间窗口）**：`10 seconds`
- Action（超限动作）
  - 测试期：选 **Managed Challenge（人机验证）**，不误伤正常用户
  - 确认攻击：选 **Block（直接拦截）**Cloudflare

6. 其他默认即可

- 不需要开 “Count only specific responses”（新手先不开）
- 点 **Save and Deploy（保存并部署）**Cloudflare

**效果**：任何 IP 在 10 秒内请求 `/login`超过 10 次，直接被 Cloudflare 屏蔽。

### 3. 启用 Bot Fight Mode

![image-20260601153817823](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601153817823.png)

**作用**：自动识别并阻止已知的恶意机器人指纹，防止黑客使用常规的攻击工具扫描你的登录口。

------

## 二、 架构层：Nginx 兜底防护

即使 CF 拦住了 99%，剩下的 1% 也可能通过某种方式漏进来。我们需要在 Nginx 上再设一道关卡。

### 1. 限制登录接口的并发与速率

![image-20260601154130955](https://ruiyeclub.oss-cn-shenzhen.aliyuncs.com/picgo/image-20260601154130955.png)

不要限制全站，只限制登录接口，避免影响其他业务。

```
http {
    # 定义一个名为 "login_limit" 的区域，限制每个 IP 每秒 1 个请求
    limit_req_zone $binary_remote_addr zone=login_limit:10m rate=1r/s;

    server {
        listen 443 ssl;
        server_name yourdomain.com;

        location = /login { # 精确匹配 login 接口
            # 应用限流，burst=5 表示允许突发 5 个请求，nodelay 表示超出的直接拒绝
            limit_req zone=login_limit burst=5 nodelay;
            
            # 限制连接数（可选）
            limit_conn perip 3; 
            
            proxy_pass http://your_springboot_app;
            # ... 其他代理配置
        }
    }
}
```

- **注意**：这里的 `rate=1r/s`非常严格，如果你们业务允许，可以调整为 `5r/s`。

### 2. 封禁异常 UA 和空 Referer

很多攻击脚本不带浏览器标识。

```
if ($request_uri ~* "/login") {
    if ($http_user_agent ~* "curl|wget|python|java|okhttp") {
        return 403;
    }
    # 如果是 POST 请求且 Referer 为空（常见于直接发包攻击）
    if ($request_method = POST) {
        set $block "";
        if ($http_referer = "") {
            set $block "${block}1";
        }
        if ($block = "1") {
            return 403;
        }
    }
}
```

------

## 三、 SpringBoot 代码层：防止“漏网之鱼”击穿数据库

当攻击流量穿透了 CF 和 Nginx，你的 SpringBoot 应用必须具备自我保护能力，防止数据库被打挂。

### 1. 登录接口防刷（Redis + 滑动窗口）

不要只依赖 Spring Security 的失败次数限制，那太重了。用 Redis 做轻量级限流。

```
@Component
public class LoginLimitInterceptor implements HandlerInterceptor {

    @Autowired
    private StringRedisTemplate redisTemplate;

    private static final int MAX_ATTEMPTS = 5; // 最大尝试次数
    private static final int LOCK_TIME_MINUTES = 15; // 锁定时长

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String ip = request.getRemoteAddr(); // 注意：如果用了CF/Nginx，这里要取 X-Forwarded-For
        String key = "login:limit:" + ip;

        Long count = redisTemplate.opsForValue().increment(key, 1);
        if (count == 1) {
            // 设置过期时间，形成滑动窗口
            redisTemplate.expire(key, LOCK_TIME_MINUTES, TimeUnit.MINUTES);
        }

        if (count > MAX_ATTEMPTS) {
            response.setStatus(429); // Too Many Requests
            response.getWriter().write("Too many login attempts, please try again later.");
            return false;
        }
        return true;
    }
}
```

### 2. 强制验证码（CAPTCHA）

当检测到某 IP 登录失败次数过多时，强制弹出验证码。

- 

  **推荐**：使用 **Google reCAPTCHA v3**（无感验证）或 **极验（Geetest）**。

- 

  **逻辑**：在 Controller 中，如果 Redis 记录该 IP 失败次数 > 3，要求前端必须携带验证码 token 才能提交登录。

### 3. 账户锁定机制

在 Spring Security 中配置：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .anyRequest().authenticated()
            .and()
        .formLogin()
            .and()
        .sessionManagement()
            .maximumSessions(1) // 限制同一账号只能一处登录
            .maxSessionsPreventsLogin(true);
}
```

并且数据库设计上要有 `account_non_locked`字段，连续输错密码 N 次，锁定账户 15 分钟。

------

## 四、 总结：Login 接口防御清单

| 层级               | 防御手段                        | 核心目的                           |
| ------------------ | ------------------------------- | ---------------------------------- |
| **Cloudflare**     | **Under Attack Mode**           | 瞬间挡住脚本流量，争取配置时间     |
| **Cloudflare WAF** | **Rate Limiting (针对 /login)** | 精准限制登录频率，这是最有效的手段 |
| **Nginx**          | **limit_req (1r/s)**            | 防止 CF 规则失效后的最后兜底       |
| **SpringBoot**     | **Redis 防刷 + 验证码**         | 防止攻击绕过代理直接打源站         |
| **系统**           | **仅允许 CF IP 访问**           | 让黑客找不到你的真实服务器         |

**最后叮嘱**：

黑客攻击登录接口往往是为了撞库（暴力破解密码）。除了限流，**务必开启 Spring Security 的密码加密（BCrypt）**，并定期提醒用户修改弱密码。只要你的登录接口响应够慢（因为限流）且密码足够复杂，黑客很快就会失去耐心转战下一个目标。

------

