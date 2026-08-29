# Nginx 反向代理与负载均衡详解：构建高性能 Web 架构的核心利器

在现代 Web 应用架构中，Nginx 作为一款高性能的 HTTP 服务器和反向代理服务器，被广泛应用于流量分发、安全防护、静态资源服务以及负载均衡等场景。其中，**反向代理（Reverse Proxy）** 和 **负载均衡（Load Balancing）** 是 Nginx 最核心、最常用的功能之一。本文将深入讲解这两个概念的原理、配置方法及最佳实践，帮助你构建高可用、可扩展的 Web 服务架构。

---

## 一、什么是反向代理？

### 1.1 正向代理 vs 反向代理

- **正向代理**：客户端通过代理服务器访问外部资源（如公司内网用户通过代理访问互联网）。代理对客户端可见，对目标服务器透明。
- **反向代理**：客户端请求发送到代理服务器，由代理服务器将请求转发给后端真实服务器，并将响应返回给客户端。客户端不知道真实服务器的存在，只与代理交互。

### 1.2 反向代理的优势

- **隐藏后端架构**：保护内部服务器不直接暴露在公网。
- **SSL 终止**：在 Nginx 上统一处理 HTTPS 加密/解密，减轻后端压力。
- **缓存加速**：缓存静态内容，减少后端请求。
- **统一入口**：便于实现日志记录、访问控制、限流等策略。

### 1.3 Nginx 反向代理基本配置

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_server;  # 转发到后端
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> 注意：`proxy_set_header` 用于将客户端原始信息传递给后端，避免后端获取到的是 Nginx 的 IP。

---

## 二、什么是负载均衡？

当单台服务器无法承载高并发请求时，我们需要将流量分发到多台服务器上，这就是**负载均衡**。Nginx 内置了多种负载均衡算法，可轻松实现横向扩展。

### 2.1 负载均衡的基本原理

Nginx 作为负载均衡器，接收客户端请求，并根据配置的策略将请求分发给一组后端服务器（称为“上游服务器”或 upstream servers），从而提高系统整体吞吐量和可用性。

### 2.2 定义 upstream 块

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

然后在 `location` 中使用：

```nginx
location / {
    proxy_pass http://backend;
}
```

---

## 三、Nginx 支持的负载均衡策略

### 3.1 轮询（Round Robin）— 默认策略

依次将请求分配给每个服务器。如果某台服务器宕机，Nginx 会自动跳过（需配合健康检查）。

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

### 3.2 加权轮询（Weighted Round Robin）

为不同服务器分配权重，性能更强的服务器处理更多请求。

```nginx
upstream backend {
    server backend1.example.com weight=3;
    server backend2.example.com weight=1;
}
```
> 表示 backend1 处理约 75% 的请求，backend2 处理 25%。

### 3.3 IP 哈希（ip_hash）

根据客户端 IP 地址的哈希值决定转发到哪台服务器，保证同一客户端始终访问同一后端（适用于会话保持场景）。

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

> ⚠️ 注意：使用 `ip_hash` 时不能设置 `weight`，且 NAT 环境下多个用户可能共享同一个公网 IP，导致负载不均。

### 3.4 最少连接（least_conn）

将请求分配给当前活跃连接数最少的服务器，适合处理长连接或耗时任务。

```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 3.5 第三方模块：一致性哈希（consistent_hash）

通过 `nginx-sticky-module` 或 `ngx_http_upstream_consistent_hash` 模块实现更高级的会话保持或缓存友好分发（需编译安装）。

---

## 四、健康检查与故障转移

Nginx 开源版本**不支持主动健康检查**，但可通过以下方式实现简单容错：

### 4.1 被动健康检查（默认行为）

- 如果某台后端服务器返回错误（如 502、504），Nginx 会暂时将其标记为不可用（`fail_timeout` 内不再转发请求）。
- 配置参数：
  ```nginx
  upstream backend {
      server backend1.example.com max_fails=3 fail_timeout=30s;
      server backend2.example.com max_fails=3 fail_timeout=30s;
  }
  ```
  > 表示连续 3 次失败后，30 秒内不再使用该服务器。

### 4.2 主动健康检查（需 Nginx Plus 或第三方模块）

商业版 Nginx Plus 提供 `health_check` 指令，可定期探测后端状态。开源用户可借助 `nginx_upstream_check_module` 等第三方模块实现。

---

## 五、实战示例：搭建高可用 Web 服务

假设我们有三台应用服务器（192.168.1.10~12），运行相同的应用，监听 8080 端口。

### 完整配置示例：

```nginx
upstream app_servers {
    least_conn;
    server 192.168.1.10:8080 max_fails=2 fail_timeout=10s;
    server 192.168.1.11:8080 max_fails=2 fail_timeout=10s;
    server 192.168.1.12:8080 max_fails=2 fail_timeout=10s;
}

server {
    listen 80;
    server_name myapp.com;

    location / {
        proxy_pass http://app_servers;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # 静态资源由 Nginx 直接提供（可选优化）
    location ~* \.(jpg|jpeg|png|css|js)$ {
        root /var/www/static;
        expires 1y;
    }
}
```

---

## 六、最佳实践建议

1. **合理设置超时时间**  
   ```nginx
   proxy_connect_timeout 5s;
   proxy_send_timeout 10s;
   proxy_read_timeout 10s;
   ```

2. **启用 Gzip 压缩** 减少传输体积：
   ```nginx
   gzip on;
   gzip_types text/plain application/json application/javascript;
   ```

3. **日志记录真实 IP**  
   使用 `$http_x_real_ip` 或 `$http_x_forwarded_for` 记录客户端 IP。

4. **结合 Keepalived 实现 Nginx 高可用**  
   避免单点故障，部署主备 Nginx + VIP。

5. **监控与告警**  
   通过 Prometheus + nginx-vts-module 监控 upstream 状态。

---

## 七、总结

Nginx 的反向代理与负载均衡功能是构建现代 Web 架构的基石。它不仅提升了系统的性能与可扩展性，还增强了安全性和运维灵活性。通过合理配置 upstream 策略、健康检查机制和请求头传递，你可以轻松应对高并发、高可用的业务需求。

无论你是运维工程师、后端开发者，还是系统架构师，掌握 Nginx 的这些核心能力，都将为你的技术栈增添强大武器。

> **小贴士**：生产环境中务必进行压力测试（如使用 wrk、ab 工具），验证负载均衡效果和故障转移能力。

---

**延伸阅读**：
- [Nginx 官方文档 - HTTP Load Balancing](https://nginx.org/en/docs/http/load_balancing.html)
- 《Nginx 高性能 Web 服务器详解》
- 使用 Docker 快速搭建 Nginx + 多后端测试环境

希望这篇博客能帮助你深入理解并高效使用 Nginx 的反向代理与负载均衡功能！
