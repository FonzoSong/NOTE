#### 分别通过用户级别、系统级别、配置文件来设置 RabbitMQ 服务的最大连接数

##### 用户级

更改配置文件：`/etc/security/limits.conf` 添加两行

```conf
rabbitmq soft nofile 10240
rabbitmq hard nofile 10240
```

##### 系统级

更改配置文件：`/etc/sysctl.conf` 添加一行
```conf
fs.filemax = 102400
```
应用更改
```bash
sysctl -p
```

##### 配置文件级

更改配置文件: `/usr/lib/systemd/system/rabbitmq-server.service`
```conf
在[Service]下添加一行参数如下：
LimitNOFILE=10240
```

应用更改
```bash
systemctl daemon-reload
systemctl restart rabbitmq-server
```
