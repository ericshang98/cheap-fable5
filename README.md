# cheap-claude

**Stop burning your Opus/Fable quota on `grep`.**

A Claude Code skill that routes work to the cheapest model that can do it. Invoke it with a task — it plans, splits the task into subtasks, and dispatches each one to a subagent on a difficulty-matched model:

| Difficulty | Model | Typical subtasks |
|---|---|---|
| Mechanical / search | `haiku` | file search, bulk code reading, running builds & checks |
| Regular | `sonnet` | feature implementation, bug fixes, tests, docs |
| Hard | `fable` (or `opus`) | architecture design, root-cause analysis of nasty bugs, complex algorithms |

The main loop stays on your cheap model; expensive models are only spun up for the subtasks that genuinely need them — chosen by the agent itself, per task.

## Why

- Claude subagents default to `inherit` — without explicit routing, **every** subtask burns your main (expensive) model.
- On Max plans, Sonnet has its own weekly limit pool separate from "All models". Routing day-to-day work to Sonnet/Haiku keeps your Opus/Fable budget for the work that deserves it.
- Hard thinking is a small fraction of most coding tasks. Pay frontier prices for that fraction only.

## Install

```bash
git clone https://github.com/ericshang98/cheap-claude.git
mkdir -p ~/.claude/skills
cp -r cheap-claude/skills/cheap-claude ~/.claude/skills/
```

That's it — it's a personal skill, available in every project. (Restart Claude Code or start a new session if it doesn't show up immediately.)

To install for a single project instead, copy it to `<project>/.claude/skills/`.

## Usage

In any Claude Code session:

```
/cheap-claude add dark mode to the Android app: settings toggle UI, theme token refactor across 40 files, global color replacement, build verification
```

The agent will:

1. Split the task into subtasks and tag each with a difficulty
2. Show you a routing table (subtask → model → why), then execute without waiting
3. Dispatch each subtask to a subagent with an **explicit `model` parameter** — independent subtasks run in parallel
4. Keep the main loop for merging results and reporting only

Pro tip: run your main session on a cheap model too —

```
/model sonnet
```

## Customize

Edit `~/.claude/skills/cheap-claude/SKILL.md`:

- No Fable 5 access? Change `fable` to `opus` in the routing table.
- Want it stingier? The skill already says "default to `sonnet`, escalate only when thinking is harder than typing" — tighten or relax that line.

## How it works

Claude Code resolves a subagent's model in priority order: `CLAUDE_CODE_SUBAGENT_MODEL` env var → per-invocation `model` parameter → subagent frontmatter → main conversation's model (`inherit`). This skill forces the per-invocation parameter on every dispatch, so nothing silently falls through to your expensive main model. See the [model config docs](https://code.claude.com/docs/en/model-config).

## License

MIT

---

## 中文说明

一个 Claude Code skill：调用时自动规划任务、按难度拆解，把子任务派给不同模型的 subagent 执行——检索杂活给 `haiku`，常规实现给 `sonnet`，只有"想清楚比写出来更难"的部分才给 `fable`/`opus`。主循环保持便宜，贵模型只为真正难的部分付费。

**安装**：clone 本仓库，把 `skills/cheap-claude` 拷到 `~/.claude/skills/`。

**使用**：任意 session 里 `/cheap-claude <你的任务>`，建议配合 `/model sonnet` 把主循环也降下来。

**为什么省钱**：subagent 不显式指定 model 时默认继承主模型——这意味着默认情况下所有子任务都在烧贵模型额度。本 skill 强制每次派发都显式指定难度匹配的模型；在 Max 订阅上 Sonnet 还有独立周限额池，日常活走 Sonnet 等于给 Opus/Fable 额度续命。
