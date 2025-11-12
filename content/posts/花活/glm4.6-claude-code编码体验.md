+++
author = "cvooc"
title = "glm4.6搭配claude-code编码体验"
date = "2025-10-16 10:35:49"
description = "glm4.6搭配claude-code编码体验"
tags = [
    "花活",
    "AI",
    "claude-code",
    "vibe coding",
    "glm"
]
+++

# glm4.6+claude-code编码体验

## 说明

一直没有深入体验过**vibe coding**, 最近没忍住开了下glm的包年套餐. 借此机会体验下现在ai编码可以进行到什么程度了.

## 环境配置

- 个人喜好原因,操作系统为**windows**
- 前置安装 **NodeJs18+**
- 进入命令行界面,安装ClaudeCode(**以下简称cc**)

```shell
npm install -g @anthropic-ai/claude-code
```

- 运行命令,查看安装结果,若显示版本号则表示安装成功

> 安装成功后, 还需后续步骤, 若直接使用 claude 命令启动, 可能由于网络或地区限制无法使用(anthropic给懂王表忠心,懂的都懂).

```shell
# 查看版本
claude --version
# 升级到最新版本
claude update
```

- 注册平台账号, 购买套餐并获取apikey, 此处我使用的是[智谱清言](https://bigmodel.cn/claude-code)的编码套餐.
- 配置环境变量

```shell
# 在 PowerShell 中运行以下命令
# 注意替换里面的 `your_zhipu_api_key` 为您获取到的 API Key
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_AUTH_TOKEN', 'your_zhipu_api_key', 'User')
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_BASE_URL', 'https://open.bigmodel.cn/api/anthropic', 'User')
[System.Environment]::SetEnvironmentVariable('CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC', '1', 'User')
```

- 进入一个代码工作目录, 在终端中执行 claude 命令即可开始使用 Claude Code
- 添加 [context7](https://github.com/upstash/context7) MCP, 用于供ai获取最新的框架文档. 使用方式也简单在提示里添加 use context7 即可.

```shell
创建一个基础vue3+ts+tsx项目. use context7
```

- 验证命令为: **claude mcp list**
- 用量查看工具 [ccusage](https://github.com/ryoppippi/ccusage), 核心推荐命令为 **ccusage blocks** 以用来查看5小时额度, glm速率间隔是5小时.

```shell
# 全局安装
npm install -g ccusage
# 常用命令
ccusage                      # = ccusage daily(默认日汇总)
ccusage daily --since 20241201 --until 20241231
ccusage weekly               # 周汇总
ccusage monthly              # 月汇总
ccusage session              # 按会话(对话)查看
ccusage blocks               # 按 5 小时计费窗查看(含 Active/预测等)
ccusage blocks --live        # 每1s实时刷新, 享受观看API Token消耗的快乐
ccusage blocks --active      # 只看当前活跃窗口
ccusage blocks --recent      # 近几天窗口
ccusage daily --breakdown    # 分模型拆解
ccusage monthly --json > usage.json  # 导出 JSON 做二次分析
```

## 使用技巧

### yolo模式启动

如果你初次使用 Claude Code, 你会发现 cc的每一条命令都需要在[ yes/no/yes并且别在问我 ]三个选项中选择并确认.
在经过初期的学习后你逐渐对此感到厌烦, 并希望它无人值守自动干活. 这时就可以启用yolo模式运行cc.

**注意!这非常危险,使用时一定要小心.因为yolo模式意味着ai会自动执行所有命令!一定注意不要使用来历不明的mcp!**

**建议在容器环境下使用, 通过容器的隔离和防火墙规则等安全措施, 最大程度的防止系统遭受攻击.**

有两种方式可以跳过权限审批.

#### 方法一：使用命令行参数

在执行命令时, 添加 **--dangerously-skip-permissions** 参数, 可以在单次执行中跳过审批.

```bash
claude --dangerously-skip-permissions
```

#### 方法二：修改配置文件

如果希望永久开启此模式, 可以修改其 **settings.json** 配置文件.添加以下内容后, cc将默认跳过所有权限提示.

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

说明：**"defaultMode": "bypassPermissions"** 指示cc将默认的操作模式设置为“绕过权限”, 从而实现自动执行.

### claude.md

claude.md是一个特殊文件, Claude在开始对话时会自动将其拉入上下文.
cc会默认读取项目根目录的 claude.md(称之为记忆), 可以把项目的核心概念/规范/TODO写在此处.
启动时, cc会**递归向上**从当前目录开始查找 CLAUDE.md 或 CLAUDE.local.md(目前主流是 CLAUDE.md)文件,并把这些内容作为"记忆"或"上下文"读入.
如果某些子目录(子树)也有 CLAUDE.md,在进入这些子目录读取这些子树时,这些子树中的 CLAUDE.md 也会被包含进上下文. 它不会在启动时就预先载入它们.

这使其成为记录以下内容的理想场所：

- 常用bash命令
- 核心文件和实用函数
- 代码风格指南
- 测试说明
- 代码库规范(例如, 分支命名、合并与变基等)
- 开发环境设置(例如, pyenv使用、哪些编译器有效)
- 项目特有的任何意外行为或警告
- 您希望Claude记住的其他信息

**CLAUDE.md文件没有要求的格式.我们建议保持简洁且人类可读.**

> 在cc终端内输入 # + 内容. 可以快捷将内容更新入 claude.md 内.

#### 引入其他md内容

claude.md 文件可以使用 @path/to/file.md 语法导入其他文件,以把说明或配置拆成多个文件.

**为了避免冲突或安全隐患, 导入语法在 Markdown 的代码块或反引号（ ``` 或 `）中不会被解释为导入**

```markdown
// 相对路径和绝对路径都可以
- 项目介绍 @docs/instructions.md
- 个人偏好设置 @~/.claude/my-project-instructions.md
- 加上``后, 不被视为导入：`@anthropic-ai/claude-code`
```

#### 核心开关 & 行为约束

官方强调把"规则/分工/节奏"写进 CLAUDE.md 并不断迭代;**将其视作高频提示词而非一次性说明书.**

```markdown
# Project Operating Mode

- Goal: 完成【<你的长程任务名称>】, 在**不中断**的前提下, 充分并行子任务, 直到产出完整、可验证的交付物.
- Autonomy: 允许在本仓库内创建/修改/删除文件；禁止写入 `node_modules`, `.git`, `dist`, `build`, `~` 与上级目录.
- Parallelism: 同时运行 ≤ <N> 个并行任务(可弹性：队列剩余任务排队).
- Edit Policy: 默认使用"Auto-accept plan"(先给出计划, 随后自动执行, 无需逐步确认).
- Checkpointing: 每轮重大变更后, **必须**更新 `PLAN.md` 与 `TASKS.md`, 以便断点续跑.
- Validation First: 任何子任务完成后, 都要跑对应校验(构建/测试/脚本), 不通过则自我修复后再进入下一个任务.
```

#### 并行调度与任务分片

利用 Task/子代理实现真正并行, 并按并发上限运行与排队.

```markdown
# Parallel Work Orchestration

## Work Decomposition
- 将目标拆成可并行的最小工作单元(WUs), 每个 WU ≤ 20 分钟理想工时.
- 对每个 WU 明确：输入、产出、校验方式、完成定义(DoD).

## Parallel Policy
- `MaxParallel = <N>`：任意时刻最多并行 N 个 WU.
- 优先级：阻塞链路优先(解开关键路径), IO/计算重任务交错布置.
- 队列：若待办 > N, 则将其入队, 先短任务、后长任务(SJF 优先策略可加速整体吞吐).

## Subagents / Tasks
- 对每个 WU 启动一个"任务 Agent"(Task), 保持相互独立的上下文与日志文件：`/logs/task-<id>.md`.
- 每个 Task 都必须在结束时写入：
  - `RESULTS/<id>.md`(产出与结论)
  - `PATCHES/<id>.diff`(文件变更集)
  - 更新 `TASKS.md` 的状态(TODO → DOING → DONE)
```

#### 长程任务"不断点"运行策略(自我复位 + 断点续跑)

将"可恢复的工作清单"写进项目文件,让模型每次启动都能自发现上下文并续跑.

```
# Long-Run Resilience

## Checkpoint Files
- `PLAN.md`: 总体路线图 + 关键里程碑 + 风险与对策(每次大改必须更新)
- `TASKS.md`: 任务清单(含优先级、预计时长、并行槽位分配、状态)
- `RISKS.md`: 风险项台账(触发条件、预案、回滚点)
- `METRICS.md`: 每轮产物质量指标(测试覆盖率、构建时长、性能基线等)

## Resume Protocol (when session restarts)
1) 打开 `PLAN.md` / `TASKS.md`, 识别未完成项与最新 Checkpoint.
2) 如果检测到构建/测试红灯, 先进入"修复模式", 直至绿灯.
3) 恢复并行队列：按 `TASKS.md` 重新分配 `MaxParallel` 槽位, 继续执行.
```

#### 质量闸口(构建/测试/验收定义)

长程并行任务若无清晰 DoD,容易"并行空转"或积累技术债.

```
# Validation & Definition of Done (DoD)

- 对每个 WU 必须定义可执行的验收脚本(例如 `npm run test:<scope>`、`pytest -k <marker>`).
- 进入下一任务前, 必须满足：
  - 构建通过
  - 测试通过(新增测试覆盖新代码路径)
  - Lint/Format 通过
  - 若涉及接口/前端：提供最小可运行 Demo 或截图/录屏证据, 附在 `RESULTS/<id>.md`.
- 不通过→自修复(最多 2 次), 仍失败则降级/回退并记录在 `RISKS.md`.
```

#### 权限与"自动批准"实践(减少打断)

```
# Permissions & Safety

- 默认在本项目根目录工作；禁止访问上级目录及系统目录.
- 允许对以下路径写入：`src`, `tests`, `scripts`, `docs`, `public`, `infra`, `configs`.
- 若需写入新目录, 必须先在 `PLAN.md` 提案并解释用途, 再执行.
```

#### 组织你的仓库(方便 Agent 自管理)

把"任务编排元数据"显式文件化,Claude Code 重启即可"看文件行事",这是很多长程项目能坚持"不中断推进"的关键.

```
/docs/CLAUDE.md          ← 本文件
/docs/PLAN.md            ← 路线图 + 里程碑
/docs/TASKS.md           ← 并行任务看板(队列/状态/优先级)
/docs/RISKS.md           ← 风险与回滚点
/docs/METRICS.md         ← 质量指标与历史
/logs/task-*.md          ← 子任务日志
/RESULTS/<id>.md         ← 子任务产出说明
/PATCHES/<id>.diff       ← 变更集
```

## 推荐阅读内容

[在 Claude Code 里安装并使用 Spec-Kit](https://zhuanlan.zhihu.com/p/1959040296479339345)