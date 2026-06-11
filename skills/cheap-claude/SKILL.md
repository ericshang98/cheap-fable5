---
name: cheap-claude
description: Use when the user invokes /cheap-claude with a task, wants to save tokens/quota/usage limits, or asks to route work across cheaper models — plans the task and dispatches subtasks to subagents on difficulty-matched models (haiku/sonnet/fable).
---

# Cheap Claude — 按难度路由模型执行任务

## Overview

主循环保持便宜，重活才花贵钱。把用户的任务拆解成子任务，按难度给每个子任务派 **指定 model 的 subagent**，而不是在主循环里自己干。Subagent 不指定 model 时默认 `inherit`（= 主模型）— 那正是要避免的浪费。

## Workflow

1. **规划（在主循环做，不派 agent）**：把 `$ARGUMENTS` 的任务拆成子任务，每个标难度。
2. **展示路由表**：执行前先用下面格式给用户看一眼（不要等确认，直接继续执行）：

   | 子任务 | 模型 | 理由 |
   |---|---|---|

3. **派发**：用 Agent 工具执行每个子任务，**每次调用必须显式传 `model` 参数**。相互独立的子任务在同一条消息里并行派发。
4. **汇总**：主循环只做合并、校验、向用户汇报。不要在主循环里读大量文件或写大段代码。

## Routing Table

| 难度 | model | 典型子任务 |
|---|---|---|
| 机械/检索 | `haiku` | 文件搜索、批量读代码、收集现状、跑命令验证、格式化 |
| 常规 | `sonnet` | 普通功能实现、改 bug、写测试、文档、code review 初筛 |
| 困难 | `fable` | 架构设计、跨模块大重构的核心方案、疑难 bug 根因分析、复杂算法 |

**升级要保守**：默认 `sonnet`；只有"想清楚比写出来更难"的子任务才给 `fable`。拿不准就 `sonnet`。explore/搜索类一律 `haiku`，绝不浪费。

## Rules

- **绝不省略 model 参数**——省略 = inherit = 烧主模型额度，违背本 skill 的存在意义。
- 主循环自己不做可以派发的重活；主循环的每个 token 都按主模型计价。
- `fable` 子任务的 prompt 必须一次性给全上下文和明确的"完成标准"（贵模型来回追问最浪费）。
- 验证/收尾（跑 build、lint、确认输出）派 `haiku` 或 `sonnet`，不要让 `fable` 干杂活。

## Common Mistakes

| 错误 | 纠正 |
|---|---|
| 任务"看着简单"就在主循环直接干完 | 超过几次工具调用的活都应拆出去 |
| 所有子任务都给 fable"保险" | 违背目的；fable 只给表里"困难"一行 |
| 串行派发独立子任务 | 独立的并行发，省时间 |
| fable 子任务 prompt 写两句话 | 贵模型必须一次喂饱：背景+约束+完成标准 |
