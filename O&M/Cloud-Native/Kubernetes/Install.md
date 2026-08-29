## How to install k8s

### 1. 添加Host解析

```bash
echo 127.0.0.1  $(hostname) | sudo  tee -a  /etc/hosts 1>> /dev/null
```

---

### 2. 安装containerd

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf update -y && sudo dnf install -y containerd.io
```

---

### 3. 配置`systemd` cgroup驱动

````bash
containerd config default | sudo tee /etc/containerd/config.toml 1>> /dev/null
````
**更改配置文件：**
结合 runc 使用 systemd cgroup 驱动，在 /etc/containerd/config.toml 中设置：

````ini
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true ## 原本是false
````

### 3. 开启端口转发

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system 1>> /dev/null
```

---

### 4. 配置conatinerd代理

```bash
sudo mkdir -p /etc/systemd/system/containerd.service.d/
cat << EOF | sudo tee /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://ip:port"
Environment="HTTPS_PROXY=http://ip:port"
Environment="NO_PROXY=localhost,127.0.0.1,.yourdomain.com"
EOF
```

**启动containerd**

```bash
sudo systemctl enable --now containerd
sudo systemctl restart containerd
```

---

### 5. 关闭selinux

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

---

### 6. 开放防火墙

**master：**

```bash
# 开放 Kubernetes API 服务器端口
sudo firewall-cmd --add-port=6443/tcp --permanent
# 开放 etcd 服务器客户端 API 端口
sudo firewall-cmd --add-port=2379-2380/tcp --permanent
# 开放 kubelet API 端口
sudo firewall-cmd --add-port=10250/tcp --permanent
# 开放 kube-scheduler 端口
sudo firewall-cmd --add-port=10259/tcp --permanent
# 开放 kube-controller-manager 端口
sudo firewall-cmd --add-port=10257/tcp --permanent
# 应用更改
sudo firewall-cmd --reload
```

**work：**

```bash
# 开放 kubelet API 端口
sudo firewall-cmd --add-port=10250/tcp --permanent

# 开放 NodePort Services 端口范围
sudo firewall-cmd --add-port=30000-32767/tcp --permanent

# 应用更改
sudo firewall-cmd --reload
```

---

### 7. 安装kubelet、kubeadm、kubectl

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.32/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet --now
```

---

### 8. 创建控制平面

```bash
sudo kubeadm config images pull
sudo kubeadm init --config=kubeadm-config.yaml
```

**kubeadm-config.yaml：**

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs: {}
skipPhases:
  - addon/kube-proxy
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
controllerManager:
  extraArgs:
    allocate-node-cidrs: "true"
    cluster-cidr: "172.168.0.0/16"
```

---

### 9.配置kubectl自动补全

```bash
sudo dnf install -y bash-completion
source /usr/share/bash-completion/bash_completion
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
sudo chmod a+r /etc/bash_completion.d/kubectl
source ~/.bashrc
```

---

## cilium Network

### 1. 开放防火墙端口

**Master：**

```bash
# 开放 etcd 端口 (TCP 2379-2380)
sudo firewall-cmd --permanent --zone=public --add-port=2379-2380/tcp

# 开放 VXLAN overlay 端口 (UDP 8472)
sudo firewall-cmd --permanent --zone=public --add-port=8472/udp

# 开放健康检查端口 (TCP 4240)
sudo firewall-cmd --permanent --zone=public --add-port=4240/tcp

# 允许 ICMP 健康检查 (Type 8, Echo Request)
sudo firewall-cmd --permanent --zone=public --add-icmp-block-inversion

# 重新加载防火墙
sudo firewall-cmd --reload
```

**Work：**

```bash
# 开放 VXLAN overlay 端口 (UDP 8472)
sudo firewall-cmd --permanent --zone=public --add-port=8472/udp

# 开放健康检查端口 (TCP 4240)
sudo firewall-cmd --permanent --zone=public --add-port=4240/tcp

# 允许 ICMP 健康检查 (Type 8, Echo Request)
sudo firewall-cmd --permanent --zone=public --add-icmp-block-inversion

# 开放出口到 Master 的 etcd 端口 (TCP 2379-2380)
sudo firewall-cmd --permanent --zone=public --add-port=2379-2380/tcp

# 重新加载防火墙
sudo firewall-cmd --reload
```

### 2. Install the Cilium CLI

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

### 3. Install Cilium

```bash
cilium install --version 1.17.2
```

### 4. Validate

```bash
cilium status --wait
cilium connectivity test
```

### 5. 获取join命令（控制平面节点执行）

```bash
echo "sudo kubeadm join $(sudo grep 'server:' /etc/kubernetes/admin.conf | awk '{print $2}' | cut -d/ -f3) --token $(sudo kubeadm token create --print-join-command 2>/dev/null | awk '{print $5}') --discovery-token-ca-cert-hash sha256:$(sudo openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | sudo openssl rsa -pubin -outform der 2>/dev/null | sha256sum | awk '{print $1}')"
```

### 6. 添加Node（工作节点执行）

```bash
kubeadm join ApiServer:Port --token Token <control-plane-host>:<control-plane-port>   --discovery-token-ca-cert-hash sha256:Hash
```

