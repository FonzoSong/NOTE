# Python `uv` 工具快速入门手册

`uv` 是一个用 Rust 编写的极快 Python 包管理器和解析器，旨在替代 `pip` 和 `pip-tools` 的工作流，同时也可替代 `virtualenv`。以下是快速入门指南：

## 安装 `uv`

```bash
# 使用 pip 安装
pip install uv

# 或者通过官方脚本安装 (Linux/macOS)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## 基本命令

### 1. 创建虚拟环境
```bash
# 创建名为 .venv 的虚拟环境
uv venv
```

### 2. 激活虚拟环境
```bash
# Linux/macOS
source .venv/bin/activate

# Windows
.\.venv\Scripts\activate
```

### 3. 安装包
```bash
# 安装单个包
uv pip install pandas

# 安装多个包
uv pip install numpy matplotlib

# 从 requirements.txt 安装
uv pip install -r requirements.txt
```

### 4. 生成锁文件
```bash
# 生成 requirements.txt
uv pip compile requirements.in -o requirements.txt

# 生成跨平台兼容的锁文件
uv pip compile requirements.in --output-file requirements.lock
```

### 5. 使用锁文件安装
```bash
uv pip sync requirements.lock
```

### 6. 导出环境配置
```bash
# 导出当前环境所有包
uv pip freeze > requirements.txt
```

### 7. 卸载包
```bash
uv pip uninstall package-name
```

## 高级功能

### 快速依赖解析
```bash
# 快速解析并安装（无需生成中间文件）
uv pip install "fastapi[all]"
```

### 兼容性选项
```bash
# 指定 Python 版本
uv venv --python 3.11

# 生成与特定系统兼容的锁文件
uv pip compile requirements.in --python-version 3.10 --platform linux
```

### 缓存管理
```bash
# 显示缓存信息
uv cache dir
uv cache info

# 清理缓存
uv cache clean
```

## 工作流示例

### 标准工作流
```bash
# 1. 创建环境
uv venv

# 2. 激活环境
source .venv/bin/activate

# 3. 安装基础包
uv pip install pandas numpy

# 4. 导出初始配置
uv pip freeze > requirements.in

# 5. 生成锁文件
uv pip compile requirements.in -o requirements.lock

# 6. 在其他机器上同步
uv pip sync requirements.lock
```

### 替代 `pip-tools`
```bash
# 更新依赖
uv pip compile --upgrade requirements.in -o requirements.lock

# 预览更新
uv pip compile --upgrade requirements.in --dry-run
```

### 配置使用清华源

[快速设置 uv 默认源为国内镜像 | uv 中文文档](https://uv.oaix.tech/blog/2025/06/17/quickly-set-uv-package-index-is-china-mirror/)

