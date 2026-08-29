## 一. 禁止使用 IP 直接访问

**目标**: 仅允许通过域名访问,避免 IP 暴露站点。

```nginx
server {
    listen 80 default_server;
    listen 443 ssl default_server;
    server_name _;
    return 444;
}
```

说明:

- `server_name _` 匹配所有未命中域名的请求。
- `444` 为 Nginx 特有,直接断开连接,比 403 更安全。

------

## 二. 防盗链（静态资源）

**目标**: 防止图片/视频/静态资源被外站引用。

```nginx
location ~* \.(jpg|jpeg|png|gif|webp|mp4|css|js)$ {
    valid_referers none blocked forgejo.fonzosong.cn *.fonzosong.cn;
    if ($invalid_referer) {
        return 403;
    }
}
```

说明:

- `none` 允许无 Referer（如浏览器直接访问）。
- `blocked` 允许被防火墙/代理隐藏 Referer 的情况。
- 建议仅对白名单域名放行。

------

## 三. 强制 HTTPS + HSTS

```nginx
server {
    listen 80;
    server_name forgejo.fonzosong.cn;
    return 301 https://$host$request_uri;
}
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

说明:

- 防止 SSL Strip。
- `preload` 前确认所有子域均支持 HTTPS。

------

## 四. 禁用危险 HTTP 方法

```nginx
if ($request_method !~ ^(GET|POST|HEAD)$) {
    return 405;
}
```

防止:

- PUT / DELETE / TRACE / OPTIONS 滥用。

------

## 五. 基础安全响应头（强烈建议）

```nginx
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

作用:

- 防点击劫持。
- 防 MIME 嗅探。
- 降低 XSS 风险。

------

## 六. 限制请求速率（防暴力/扫描）

```nginx
limit_req_zone $binary_remote_addr zone=req_limit:10m rate=10r/s;

location / {
    limit_req zone=req_limit burst=20 nodelay;
}
```

说明:

- 有效防止暴力破解、简单 CC 攻击。
- 可对 `/login` 单独加严。

------

## 七. 隐藏 Nginx 版本信息

```nginx
server_tokens off;
```

避免泄露版本号,降低被针对性攻击概率。

------

## 八. 限制上传文件大小（防止 DoS）

```nginx
client_max_body_size 10m;
```

建议:

- API / 上传接口单独定义。
- Forgejo / Git 类服务需适当放大。

------

## 九. 禁止访问敏感文件

```nginx
location ~ /\.(env|git|svn|hg|DS_Store) {
    deny all;
}
location ~* (composer\.json|composer\.lock|package\.json|yarn\.lock)$ {
    deny all;
}
```

------

## 最小安全基线

必须有:

- 禁 IP 访问
- HTTPS + HSTS
- 安全响应头
- 防盗链
- 限速
- 禁止危险方法
- 隐藏版本号