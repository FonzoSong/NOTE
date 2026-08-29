AI Prompt Engineering 的本质不是“写提示词”，而是构建一套：

> 可版本化、可评估、可回归、可控制行为边界的模型交互系统

在生产环境中，Prompt 已经演化为一种“软代码层”，其复杂度接近配置驱动的运行时系统。

------

# 1. AI 非确定性与工程影响

## 1.1 非确定性的技术来源

LLM 输出不稳定的根因包括：

### (1) 采样机制

- temperature 控制随机性
- top-p 控制概率截断
- sampling 导致多路径生成

### (2) 神经权重分布的概率性

模型并不是“执行逻辑”，而是：

> token distribution estimation system

因此不存在严格 deterministic execution。

------

### (3) 对齐机制影响

RLHF / DPO / instruction tuning 会引入：

- “偏好输出”
- “安全拒绝行为”
- “风格扰动”

这些行为本身是统计意义上的，而不是规则驱动。

------

## 1.2 模型差异带来的工程问题

不同模型在 Prompt 执行层面差异巨大 ：

| 维度               | GPT类 | Claude类 | 开源模型 |
| ------------------ | ----- | -------- | -------- |
| instruction follow | 中-强 | 强       | 弱       |
| 长上下文稳定性     | 中    | 强       | 不稳定   |
| tool calling       | 强    | 很强     | 依赖实现 |
| format compliance  | 中    | 强       | 弱       |

### 工程结论：

> Prompt 不可跨模型直接迁移，必须做 adapter 层适配。

------

# 2. 生产级系统设计原则（核心）

## 2.1 模型快照绑定（Model Pinning）

生产系统必须避免：

- 模型版本漂移
- API behavior change
- 对齐策略更新影响输出

### 推荐策略：

- 固定 model version
- 固定 decoding 参数
- 固定 system prompt version
- Prompt 版本纳入 Git 管理

------

### 推荐架构：

```text
App Layer
   ↓
Prompt Registry (Git)
   ↓
Model Adapter Layer
   ↓
LLM Runtime (Pinned Model)
```

------

## 2.2 Prompt as Code（关键原则）

Prompt 必须具备软件工程属性：

- versioning
- diff review
- rollback
- CI test

------

## 2.3 Prompt Registry 设计

推荐结构：

```text
prompts/
  analytics/
    v1/
      system.xml
      developer.xml
    v2/
      system.xml
```

每次修改必须：

- PR review
- eval test
- regression check

------

# 3. Prompt 标准结构设计（工业级模板）

## 3.1 XML结构化是最佳实践

推荐标准结构 ：

```xml
<identity>
You are a system-level assistant specialized in ...
</identity>

<instructions>
1. Follow these rules strictly
2. Do not deviate from schema
3. Handle edge cases explicitly
</instructions>

<output_format>
Return JSON with the following schema:
...
</output_format>

<examples>
<example>
<input>...</input>
<output>...</output>
</example>
</examples>

<context>
External data / API / documents
</context>
```

------

## 3.2 为什么 XML 优于自然语言

### 原因：

- 降低 token ambiguity
- 明确 segment boundary
- 提高 instruction parsing accuracy
- 防止 instruction injection contamination

------

## 3.3 结构设计原则

### 核心原则：

- instruction 与 context 分离
- output schema 必须显式定义
- examples 必须结构化
- role 必须明确声明

------

# 4. Few-shot Learning 工程化设计

## 4.1 Few-shot 的本质

Few-shot 并不是“示例”，而是：

> behavior conditioning mechanism（行为条件化机制）

------

## 4.2 示例设计要求

必须满足：

### (1) 相关性（Relevance）

必须贴近真实输入分布

------

### (2) 多样性（Diversity）

避免单一 pattern bias

------

### (3) 覆盖边界条件

必须包含：

- normal case
- edge case
- failure case

------

## 4.3 推荐结构

```xml
<examples>
  <example>
    <input>...</input>
    <output>...</output>
  </example>
</examples>
```

------

## 4.4 示例数量建议

- 最优：3–5 个
- 超过 7 个可能导致：
  - token 浪费
  - pattern overfitting

------

# 5. 长上下文工程策略（关键能力）

## 5.1 长上下文基本问题

当输入 > 20k tokens：

- attention dilution
- query loss
- instruction overshadowing

------

## 5.2 最优结构顺序

必须遵循：

```text
Document Data (top)
→ Instructions
→ Query (bottom)
```

------

## 5.3 多文档结构化

推荐标准：

```xml
<documents>
  <document index="1">
    <source>fileA</source>
    <document_content>...</document_content>
  </document>
</documents>
```

------

## 5.4 Query placement effect

实验结论：

> Query 放在末尾可提升 20–30% 输出质量

------

## 5.5 长任务策略

- 分阶段处理
- 每阶段输出可验证结果
- 避免一次性推理

------

# 6. Tool Use（工具调用）工程控制

## 6.1 Tool trigger 是显式行为

模型不会自动执行：

- suggest ≠ execute

必须明确：

- “do it”
- “apply changes”
- “call tool”

------

## 6.2 两种控制模式

### 强执行模式

```text
If action is possible, execute directly.
```

### 保守模式

```text
Only act when explicitly instructed.
```

------

## 6.3 工具误触发问题

Claude 新模型存在：

- tool over-triggering
- agent over-aggression

解决方案：

- 减少 CRITICAL / MUST 类指令
- 使用自然语言约束

------

## 6.4 并行工具调用

支持并行执行：

- file read
- search
- API calls

策略：

```text
If independent, execute in parallel.
If dependent, execute sequentially.
```

------

# 7. Thinking / Reasoning 控制体系

## 7.1 Thinking 并非越多越好

问题：

- latency 上升
- token cost 上升
- overthinking risk

------

## 7.2 推荐策略

### (1) 使用 general instruction

替代：

- think step-by-step

推荐：

- evaluate
- reason through
- consider options

------

### (2) 自检机制（推荐）

```text
Before final answer, verify against constraints.
```

------

## 7.3 思维控制目标

- 减少无效推理
- 提升决策稳定性
- 控制 token budget

------

# 8. Agentic System（智能体系统设计）

## 8.1 核心能力

Claude 类模型支持：

- multi-step planning
- state tracking
- subagent orchestration

------

## 8.2 状态管理是核心

必须显式维护：

```json
{
  "task": [],
  "progress": "",
  "state": ""
}
```

------

## 8.3 长任务执行模式

推荐：

- 分阶段执行
- 每阶段 checkpoint
- 外部持久化 state

------

## 8.4 Git 作为状态系统

最佳实践：

- commit = checkpoint
- branch = experiment
- log = memory

------

## 8.5 子代理机制

模型可能自动生成 subagents：

### 优势：

- 并行处理
- 上下文隔离

### 风险：

- 过度拆解
- 性能浪费

控制策略：

```text
Use subagents only when tasks are independent.
```

------

# 9. 常见失败模式与修正

## 9.1 Over-engineering

表现：

- 自动增加功能
- 过度抽象

修正：

```text
Only implement what is requested.
```

------

## 9.2 Hardcoding bias

表现：

- 针对 test case 写死逻辑

修正：

```text
Do not optimize for tests. Optimize for generality.
```

------

## 9.3 Hallucination

表现：

- 未读取文件就回答

修正：

```text
Never speculate without reading source data.
```

------

## 10. 生产级 Prompt Checklist

上线前必须检查：

### 结构层

-  XML结构清晰
-  output schema 明确
-  instruction 无歧义

### 行为层

-  tool trigger 明确
-  thinking 控制策略存在
-  subagent 策略明确

### 质量层

-  few-shot 覆盖边界
-  eval test 已存在
-  regression 可回归

------

# 11. 总结（工程本质）

Prompt Engineering 的最终形态是：

> 将 LLM 从“语言生成系统”改造为“受控执行系统”

其核心三层结构：

### 1. 结构层

XML / schema / context separation

### 2. 行为层

tool / agent / thinking control

### 3. 评估层

eval / regression / CI system