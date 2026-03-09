---
name: model-router
description: 按任务复杂度路由到合适的 Claude 模型，平衡成本与速度。在每条消息上评估是否应该切换模型。使用场景：派发子任务、在昂贵模型上做简单工作、或在便宜模型上碰到复杂问题时。
---

# Model Router

用最便宜的模型搞定任务。

## 模型层级

| 级别 | 模型 | 用途 |
|------|------|------|
| **Fast** | `anthropic/claude-haiku-4.5` | 查询、格式化、翻译、简单Q&A、heartbeat、定时任务、子任务 |
| **Balanced** | `anthropic/claude-sonnet-4.6` | 写作、分析、代码、研究、产品思考、交互对话（默认） |
| **Deep** | `anthropic/claude-opus-4.6` | 架构决策、安全审查、复杂调试、高风险输出 |

## 成本参考

| 级别 | Input $/1M | Output $/1M | 相对成本 |
|------|-----------|------------|---------|
| Fast | $1.00 | $5.00 | **1x** |
| Balanced | $3.00 | $15.00 | 3x |
| Deep | $5.00 | $25.00 | 5x |

## 判断规则

**何时用 Fast（Haiku）：**
- 单步、无歧义的任务
- 无需人工判断或合成
- Cron / heartbeat / 监控任务
- 子任务做 fetch / parse / 格式化

**何时用 Balanced（Sonnet）：**
- 多步但流程清晰
- 写作、总结、代码生成
- 大多数交互对话（默认）

**何时用 Deep（Opus）：**
- 无先例的复杂问题
- Sonnet 已尝试但失败
- 高风险或不可逆操作
- 需要深层多步推理

## Per-Agent 默认值

每个 Agent 有自己的默认模型，在上述判断规则基础上升降：

| Agent | 名字 | 默认 | 职责 |
|-------|------|------|------|
| main | Jonson | Balanced | 交互对话、产品决策 |
| dev | Evan | Balanced | 代码质量需要稳定 |
| design | Ivy | Fast | 设计描述、参考收集 |
| ops | Steven | Fast | 监控、报表、格式化 |

## 行为规则

**主会话（交互）：**
- 默认 Balanced，这是你的工作模型
- 遇到简单任务 → 说"这个可以用 Haiku 节省成本，要切吗？"
- 遇到复杂问题 → 说"这个需要 Opus 的推理能力，确认吗？"

**子任务：**
- 默认 Fast，除非任务明显是 Balanced+
- 不要无理由用 Deep 派子任务

**自动降级：**
- 完成 Deep 任务后，若下一条消息是 Fast/Balanced 级别
- 主动切回 Balanced，告知："复杂任务完成，已切回 Sonnet。"

**静默模式（可选）：**
- 用户说"自动切换"或`/router silent`时
- 停止逐次确认，直接切并在回复末尾标注：`[🟢 Haiku]` / `[🟡 Sonnet]` / `[🔴 Opus]`

## 模型切换

```
/model anthropic/claude-haiku-4.5     # 切到 Fast
/model anthropic/claude-sonnet-4.6    # 切回 Balanced
/model anthropic/claude-opus-4.6      # 切到 Deep
```

## 反模式

- ❌ 用 Sonnet/Opus 跑 heartbeat 或 cron
- ❌ 用 Opus 回答简单 Q&A
- ❌ 无故用 Deep 派子任务
- ❌ 频繁在模型间切换（计个数，超过 3 次询问要不要开静默模式）
- ✅ 从 Balanced 开始，按需升降
- ✅ 自动化任务永远用 Fast
- ✅ 失败后升级，完成后降级
