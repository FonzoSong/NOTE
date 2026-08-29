# 1. Hash 的核心定义

## 1.1 通用定义

Hash（哈希）是一个函数或过程：

> 将任意长度的输入数据映射为固定长度的输出值。

数学形式：

```
H: {0,1}* → {0,1}^n
```

含义：

- 输入：任意长度数据（字符串、文件、结构体、流）
    
- 输出：固定长度摘要（hash value / digest）
    

例如：

```
H("hello") = a91b3f...
H("very large file...") = 9c02ff...
```

---

## 1.2 两个核心语义分支

Hash 在计算机中有两个主要语境：

### A. 数据结构 Hash（Hash Table）

目标：快速查找

- 用 hash 作为“索引映射”
    
- 典型结构：HashMap / Dictionary
    

### B. 密码学 Hash（Cryptographic Hash）

目标：安全性

- 防篡改
    
- 不可逆
    
- 抗碰撞
    

---

# 2. 数据结构中的 Hash（工程核心）

## 2.1 Hash Table 的本质

Hash Table =

> 数组 + 哈希函数 + 冲突处理机制

结构：

```
key → H(key) → index → array[index]
```

例如：

```
"alice" → hash → 3 → table[3] = value
```

---

## 2.2 Hash Function 在数据结构中的目标

关键指标：

- O(1) 平均查找复杂度
    
- 均匀分布（uniform distribution）
    
- 低冲突率
    

---

## 2.3 常见 Hash 方法

### 2.3.1 取模法（Modulo Hash）

```
H(x) = x mod N
```

问题：

- N 选不好会导致 clustering
    

---

### 2.3.2 Multiplicative Hash

```
H(x) = floor(N * (x * A mod 1))
```

特点：

- 分布更均匀
    
- 常用于高性能 hash table
    

---

### 2.3.3 字符串 Hash（典型）

#### Java / Go 风格：

```
h = h * 31 + c
```

例如：

```
"h" → 104
"he" → 104*31 + 101
```

特点：

- 简单
    
- 快速
    
- 易实现
    

---

## 2.4 冲突（Collision）

不同输入映射到同一输出：

```
H(x1) = H(x2), x1 ≠ x2
```

冲突不可避免（鸽巢原理）

---

## 2.5 冲突解决策略

### 2.5.1 链地址法（Chaining）

```
table[i] → linked list
```

### 2.5.2 开放寻址（Open Addressing）

- linear probing
    
- quadratic probing
    
- double hashing
    

---

## 2.6 Hash Table 性能本质

平均：

```
O(1)
```

最坏：

```
O(n)
```

（当全部冲突时退化为链表）

---

# 3. 密码学 Hash（核心重点）

## 3.1 定义

密码学 hash 是一种：

> 单向函数（one-way function）

满足：

```
H(m) → digest
```

但不可逆：

```
H⁻¹(digest) 不可计算
```

---

## 3.2 核心安全属性（必须掌握）

### 3.2.1 原像不可逆（Preimage resistance）

给定 hash：

```
y = H(x)
```

几乎不可能找到 x

---

### 3.2.2 第二原像抗性（Second Preimage）

给定 x1：

```
H(x1) = H(x2)
```

难以找到 x2

---

### 3.2.3 碰撞抗性（Collision resistance）

难以找到任意：

```
x1 ≠ x2
但 H(x1) = H(x2)
```

---

## 3.3 为什么必须“固定长度输出”

原因：

- 输入空间无限
    
- 输出空间有限
    

因此必然存在：

- 压缩
    
- 信息丢失（irreversible）
    

---

## 3.4 常见算法

|算法|状态|
|---|---|
|MD5|已破解|
|SHA-1|不安全|
|SHA-256|当前主流|
|SHA-3|新标准|

---

## 3.5 Hash vs Encryption（关键区别）

|特性|Hash|Encryption|
|---|---|---|
|可逆性|❌ 不可逆|✔ 可逆|
|key|无|有|
|用途|校验/签名/存储|传输加密|

---

# 4. Hash 的工程应用

## 4.1 密码存储

错误方式：

```
store password directly ❌
```

正确方式：

```
store H(password)
```

更安全：

```
H(password + salt)
```

---

## 4.2 文件完整性校验

下载文件：

```
file → hash → compare
```

用途：

- 防篡改
    
- CDN 校验
    
- 软件分发
    

---

## 4.3 数字签名（核心依赖）

流程：

```
message → hash → sign(hash)
```

原因：

- 签名原文太大成本高
    
- hash 作为“指纹”
    

---

## 4.4 Blockchain / Merkle Tree

区块链：

```
tx1 → hash
tx2 → hash
...
→ merkle root
```

任何修改都会导致：

- root hash 改变
    

---

# 5. Hash 的设计目标（工程视角）

一个好的 hash function 需要：

## 5.1 Avalanche Effect（雪崩效应）

输入变化 1 bit：

```
输出大规模变化
```

---

## 5.2 Uniform Distribution

输出均匀分布：

- 避免 clustering
    
- 提升 hash table 性能
    

---

## 5.3 Determinism

同样输入必须：

```
H(x) == H(x)
```

---

## 5.4 Efficiency

计算复杂度：

```
O(n)
```

但常数极小

---

# 6. 安全攻击模型

## 6.1 Brute Force（暴力破解）

尝试所有输入：

```
O(2^n)
```

---

## 6.2 Birthday Attack（生日攻击）

核心结论：

```
n-bit hash
碰撞复杂度 ≈ 2^(n/2)
```

例如：

- SHA-256 → 2^128 安全级别
    

---

## 6.3 Length Extension Attack

存在于：

- MD5
    
- SHA-1
    
- SHA-256（Merkle-Damgård结构）
    

---

# 7. Hash 在现代系统中的地位

## 7.1 基础设施级依赖

- Git commit ID
    
- Docker image digest
    
- Kubernetes image pull
    
- TLS certificate chain
    

---

## 7.2 DevOps 场景（与你相关）

常见用途：

- artifact fingerprint
    
- image immutability
    
- config integrity
    
- cache key（CI/CD）
    

例如：

```
cache key = hash(dependency.lock)
```

---

# 8. 常见误区

## 8.1 Hash ≠ 加密

hash：

- 不可解密
    

encryption：

- 可解密
    

---

## 8.2 Hash ≠ 随机数

hash：

- deterministic
    

random：

- non-deterministic
    

---

## 8.3 Hash 冲突“可以忽略”是工程假设

不是数学保证，而是：

- 概率极低
    

---

# 9. 一个统一抽象模型

可以把 hash 看作：

```
压缩函数 + 混淆函数 + 随机映射近似
```

目标：

- 压缩输入空间
    
- 保持区分性
    
- 尽量像 random oracle
    

---

# 10. 总结

Hash 本质可以压缩为一句话：

> Hash 是一种将任意长度数据映射为固定长度“指纹”的函数，在不同语境下分别用于数据结构索引优化与密码学安全保证。