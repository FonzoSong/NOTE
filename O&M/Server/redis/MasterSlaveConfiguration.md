### 参考文档:
https://redis.io/docs/latest/operate/oss_and_stack/management/replication/
https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/#fundamental-things-to-know-about-sentinel-before-deploying
### redis.conf 更改项

master node
```conf
bind 0.0.0.0 # 允许任何连接
protected-mode no # 关闭保护模式
port 6379 # 暴露端口
daemonize yes
cluster-enabled no # 不启用 集群
appendonly yes # 不覆盖日志 会导致日志文件过大 部署成功后改为 no
requirepass 123456 # 客户端访问密码
```

slave node
```conf
bind 0.0.0.0 # 允许任何连接
daemonize yes
protected-mode no # 关闭保护模式
port 6379 # 暴露端口
cluster-enabled no # 不启用 集群
appendonly yes # 不覆盖日志 会导致日志文件过大 部署成功后改为 no

# redis 5
slaveof 192.168.100.11 6379

# redis 6
replicaof 192.168.100.11 6379

masterauth 123456 # 客户端访问密码
```

### sentinel.conf 更改项

```conf
bind 0.0.0.0
daemonize yes
port 26379
sentinel monitor mymaster 192.168.100.11 6379 2
protected-mode no
```
