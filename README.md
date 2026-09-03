# unattended-task-pitfalls

**无人值守自动任务加固经验库——"操作纪律会失效，只有代码能兜底"。来自两次备份损坏事故 + cron 总结误判事故的完整复盘。**

**A hardening playbook for AI-agent unattended/scheduled tasks — "operational discipline fails unattended; only code has your back." Distilled from two real backup-corruption incidents and a cron summarization misjudgment.**

[中文](#中文) | [English](#english)

---

## 中文

### 这是什么

一个 AI agent 工作区的真实事故复盘（2026-08）：

- **08-27**：定时备份产出损坏文件并上传云端，教训"gzip -t 验证后才能上传"被记入长期记忆
- **08-30**：同款事故**复发**——因为 cron 会话根本读不到"操作纪律"，纪律只对在线的 agent 有效。528MB 截断坏文件再次上传（正常 911MB）
- **08-24/25**：cron 总结任务在隔离会话里误判"全天无对话"（它看不到主会话上下文）

由此提炼出本库的核心命题：**无人值守链路里，操作纪律会失效，只有代码能兜底。**

### 内容结构

| 章节 | 解决什么 |
|------|---------|
| §1 核心哲学 | 为什么"记忆型教训"在无人值守场景必然失效，教训落点分级（脚本内 > prompt > 记忆） |
| §2 后台执行 | 前台跑被网关杀、"504 ≠ 命令未执行"（盲目重试 = 并发实例互踩）、锁与幂等 |
| §3 验证链 | 产物完整性验证（`gzip -t`）、"验证不过绝不上传"闸门、"已存在跳过"复用坏文件坑 |
| §4 隔离会话 | cron 会话的上下文盲区、"无对话/静默"结论的证据法（含会话初始化产物的排除清单） |
| §5 通知设计 | cron 无条件通知 vs 心跳条件通知、自动任务输出禁用拟人内容、静默档防通知疲劳 |
| §6 交叉验证 | 自动任务的产物也要被自动检查（发现 08-30 事故的唯一渠道）、一次性任务自删、失败的默认出口 |

### 核心原则速览

1. 凡无人值守的纪律，**一次性落到代码与 prompt**（锁、验证闸门、复查脚本），不要指望 agent"想起来"
2. 单一任务的自我报告不可信——给关键产物配**独立验证路径**
3. 网关/管道类超时 ≠ 命令未执行，**先查进程再决定重试**
4. 失败的默认出口 = 记录日志 + 通知用户 + 保留现场，不在失败路径上做主观决策

### 适合谁

- 给 AI agent（Claude Code / OpenClaw / 自建 agent）搭建 cron / 定时备份 / 自动报告的人
- 遇到过"任务失败但报告成功"、"备份文件损坏"、"定时总结瞎编"的人
- 设计多任务自动化体系、需要通知与静默策略的人

### 安装

```bash
git clone https://github.com/mowenQWQ/unattended-task-pitfalls.git
cp -r unattended-task-pitfalls /path/to/your/agent/skills/
```

`scripts/` 内含产物验证脚本模板（`verify_archive.py`），可直接适配到你的备份任务。

### 更多技能库

| 仓库 | 简介 |
|------|------|
| [win-dev-pitfalls](https://github.com/mowenQWQ/win-dev-pitfalls) | Windows 开发踩坑全量主线 |
| [bat-ps1-dev](https://github.com/mowenQWQ/bat-ps1-dev) | bat/PowerShell 脚本开发专项经验库 |
| [Web-Security-Test-Rules](https://github.com/mowenQWQ/Web-Security-Test-Rules) | 已授权网站安全测试规则库 |
| [agent-mistake-patterns](https://github.com/mowenQWQ/agent-mistake-patterns) | AI 智能体犯错模式库与自我纠错纪律 |
| [grounded-summaries-skill](https://github.com/mowenQWQ/grounded-summaries-skill) | 防止 AI 在总结任务中编造内容 |

---

## English

### What is this

A post-mortem from a real AI-agent workspace (August 2026):

- **08-27**: a scheduled backup uploaded a corrupted archive; the lesson ("verify with `gzip -t` before upload") was stored in long-term memory
- **08-30**: the **same incident recurred** — because cron sessions never read "operational discipline"; discipline only binds an online agent. Another truncated 528MB file (expected: 911MB) went upstream
- **08-24/25**: a cron summarization task in an isolated session misjudged "no conversations today" (it cannot see the main session's context)

The core thesis: **in unattended pipelines, operational discipline fails — only code has your back.**

### Structure

| Section | Covers |
|---------|--------|
| §1 Core philosophy | Why "memory-stored lessons" inevitably fail unattended; lesson-placement tiers (in-script > prompt > memory) |
| §2 Background execution | Gateway-killed foreground runs, "504 ≠ not executed" (blind retries = concurrent instances racing), locks & idempotency |
| §3 Verification chain | Artifact integrity (`gzip -t`), "never upload without verification" gates, the "skip if exists" reuse-of-corrupt-file trap |
| §4 Isolated sessions | Cron context blindness; evidence-based "no activity" conclusions (with init-artifact exclusion list) |
| §5 Notification design | Unconditional cron vs conditional heartbeat notifications; no anthropomorphic filler in automated output; silence tiers |
| §6 Cross-verification | Automated artifacts deserve automated checks; one-shot task self-deletion; default failure exits |

### Quick principles

1. Move every unattended discipline **into code and prompts once** (locks, verification gates, review scripts) — don't count on the agent "remembering"
2. Never trust a task's self-report — give critical artifacts an **independent verification path**
3. Gateway/pipeline timeouts ≠ command not executed — **check the process before retrying**
4. Default failure exit = log + notify + preserve the scene; make no subjective decisions on the failure path

### Installation

```bash
git clone https://github.com/mowenQWQ/unattended-task-pitfalls.git
cp -r unattended-task-pitfalls /path/to/your/agent/skills/
```

`scripts/` includes a reusable artifact-verification template (`verify_archive.py`).

---

## License

MIT
