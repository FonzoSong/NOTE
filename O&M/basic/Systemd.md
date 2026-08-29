# 服务单元文件编写指南

## 1.什么是Systemd服务单元文件？

Systemd 是现代 Linux 发行版的主要初始化系统和服务管理器。服务单元文件（Service Unit File）用于定义和管理 Systemd 服务。

服务单元文件通常存放在：

- **系统级**：`/etc/systemd/system/`
- **用户级**：`~/.config/systemd/user/`

文件格式：`*.service`

---

## 2.服务单元文件的基本结构

服务单元文件一般由三个主要部分组成：

1. `[Unit]` —— 描述服务的元数据和依赖关系
2. `[Service]`——配置具体的服务行为
3. `[Install]`——定义如何启用服务

---

## 3.示例

下面是一个 Systemd 服务单元文件示例（`myapp.service`），用于管理一个名为 `myapp` 的应用：

```ini
iniCopyEdit[Unit]
Description=MyApp Service  # 服务描述
After=network.target       # 依赖网络服务启动后再启动

[Service]
Type=simple                # 进程类型
ExecStart=/usr/local/bin/myapp  # 启动命令
Restart=always             # 失败时自动重启
User=myuser                # 运行用户
Group=mygroup              # 运行用户组
WorkingDirectory=/opt/myapp # 工作目录
Environment="ENV_VAR=value" # 设置环境变量
LimitNOFILE=10240          # 限制文件描述符数量
ExecStop=/bin/kill $MAINPID # 终止命令

[Install]
WantedBy=multi-user.target # 设定服务运行的目标环境
```

---

## **4. 关键参数解析**

### **[Unit] 部分**

| 参数           | 作用                                     |
| -------------- | ---------------------------------------- |
| `Description=` | 服务的简要描述                           |
| `After=`       | 指定在某个服务或目标之后启动             |
| `Before=`      | 指定在某个服务或目标之前启动             |
| `Requires=`    | 依赖其他服务（必须启动，否则失败）       |
| `Wants=`       | 推荐依赖其他服务（失败不会影响自身启动） |

### **[Service] 部分**

| 参数                | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| `Type=simple`       | 默认类型，直接运行 `ExecStart` 指定的进程                    |
| `Type=forking`      | 进程以后台方式运行（如 `daemon`）                            |
| `Type=oneshot`      | 适用于一次性任务，如初始化脚本                               |
| `ExecStart=`        | 指定启动服务的命令                                           |
| `ExecStop=`         | 指定停止服务的命令                                           |
| `ExecReload=`       | 指定重新加载服务的命令                                       |
| `Restart=`          | 失败时自动重启（`no`、`always`、`on-failure`、`on-abnormal` 等） |
| `User=`             | 指定运行该服务的用户                                         |
| `Group=`            | 指定运行该服务的用户组                                       |
| `Environment=`      | 指定环境变量                                                 |
| `WorkingDirectory=` | 指定服务运行的工作目录                                       |

### **[Install] 部分**

| 参数          | 作用                                           |
| ------------- | ---------------------------------------------- |
| `WantedBy=`   | 指定服务安装到的目标（如 `multi-user.target`） |
| `RequiredBy=` | 设定被其他服务必须依赖                         |
| `Alias=`      | 设定服务的别名                                 |

------

## **5. 服务管理命令**

| 命令                                  | 作用                  |
| ------------------------------------- | --------------------- |
| `systemctl daemon-reload`             | 重新加载 Systemd 配置 |
| `systemctl start myapp`               | 启动 `myapp` 服务     |
| `systemctl stop myapp`                | 停止 `myapp` 服务     |
| `systemctl restart myapp`             | 重启 `myapp` 服务     |
| `systemctl status myapp`              | 查看 `myapp` 状态     |
| `systemctl enable myapp`              | 设为开机自启          |
| `systemctl disable myapp`             | 取消开机自启          |
| `systemctl list-units --type=service` | 列出所有服务          |

------

## **6. 进阶功能**

### **日志管理**

Systemd 使用 `journalctl` 记录日志：

```
shCopyEditjournalctl -u myapp.service  # 查看 myapp 服务日志
journalctl -xe  # 查看系统日志
```

### **限制资源**

可以使用 `LimitNOFILE=10240` 限制最大文件描述符数，或 `CPUQuota=50%` 限制 CPU 资源。

### **定时启动**

结合 `systemd-timer`，可用 `myapp.timer` 代替 `cron` 进行定时任务管理。