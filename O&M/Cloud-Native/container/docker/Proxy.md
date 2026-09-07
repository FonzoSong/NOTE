### 设置系统全局代理
在 `～/.bashrc` 中添加
```bash
export http_proxy="http://your_proxy_address:port"
export https_proxy="http://your_proxy_address:port"
```
#### 虚拟机中的 `docker`
编辑 /etc/environment：

```bash
vim /etc/environment
```
然后添加以下内容：

```bash
http_proxy="http://your_proxy_address:port"
https_proxy="http://your_proxy_address:port"
```
保存后，重启终端或使用 source /etc/environment 使配置生效。

### docker 代理配置文件
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo vim /etc/systemd/system/docker.service.d/proxy.conf
```
``` conf
[Service]
Environment="HTTP_PROXY=http://your_proxy_address:port"
Environment="HTTPS_PROXY=http://your_proxy_address:port"
Environment="NO_PROXY=localhost,127.0.0.1"
```

### 重载服务

```bash
sudo systemctl daemon-reload 

sudo systemctl restart docker
```



