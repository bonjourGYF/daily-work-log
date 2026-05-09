# Daily Work Log — 每日工作自动记录 Skill

一个跨平台的 Claude skill，自动记录每日项目工作内容，生成结构化的 Markdown 工作日志。

**支持平台：** Cowork（Claude 桌面应用）| Claude Code（终端） | VS Code / JetBrains IDE 插件

## 功能

- **自动定时记录** — 每天早上自动汇总前一天的所有工作内容
- **手动即时记录** — 随时说"记录今天的工作"，立即生成报告
- **智能跳过** — 没有对话的日期不生成空文件
- **编号递增** — 同一天多次记录生成独立文件（-1, -2, -3...）
- **合成式总结** — 自动将所有会话合成为一段连贯的工作叙述，而非逐条罗列
- **跨平台** — Cowork、Claude Code、IDE 插件均可使用

## 工作记录内容

每份记录包含四个章节：

1. **工作概述** — 当日工作的高层总结
2. **工作内容与进展** — 具体完成的任务、编写的代码、取得的成果、做出的决策
3. **文件变更汇总** — 新建和修改的文件清单
4. **问题与笔记** — 待办事项和遇到的问题

## 环境要求

| 要求 | Cowork | Claude Code | IDE 插件 |
|------|--------|-------------|----------|
| Claude 版本 | 任意 | 任意 | 任意 |
| 额外依赖 | 无 | `crontab`（自动模式） | `crontab`（自动模式） |
| Python 包 | 无 | 无 | 无 |

- **无需任何 Python 包或第三方工具**
- Windows 用户使用自动模式时需用 Task Scheduler 替代 crontab（skill 会引导配置）

## 安装

### Cowork（Claude 桌面应用）

1. 下载 `daily-work-log.skill` 文件
2. 在 Cowork 中打开该文件，系统会自动识别安装

### Claude Code（终端）

```bash
# 克隆本仓库
git clone <repo-url>
cd daily-work-log

# 将 SKILL.md 注册为 skill
# 方法一：使用 save_skill（在 Claude Code 对话中）
# 方法二：将 daily-work-log/ 文件夹放入 Claude Code 的 skills 目录
```

### IDE 插件（VS Code / JetBrains）

在插件的 skill 管理界面导入 `daily-work-log.skill` 文件，或将 `daily-work-log/` 目录放入 skills 文件夹。

## 使用

### 设置自动记录

在任何项目的对话中说：

> "设置自动工作记录"

skill 会依次询问：
1. **项目路径** — 工作记录保存到哪个目录
2. **执行时间** — 每天几点自动记录（默认早上 6:00）
3. **确认信息** — 展示汇总配置供你确认

确认后，平台会自动创建定时任务：
- **Cowork**：通过 `scheduled_tasks` MCP 工具创建
- **Claude Code / IDE**：生成 shell 脚本 + 注册 crontab 定时任务

### 手动记录

说以下任意一句即可触发：

> "记录今天的工作"
> "总结一下今天做了什么"
> "生成昨天的工作报告"

手动模式在所有平台上行为一致——直接总结当前对话内容，无需读取外部文件。

## 各平台差异

| 功能 | Cowork | Claude Code | IDE 插件 |
|------|--------|-------------|----------|
| 会话读取（手动） | 当前对话上下文 | 当前对话上下文 | 当前对话上下文 |
| 会话读取（自动） | `session_info` MCP | `~/.claude/` 历史文件 | IDE 历史目录 |
| 定时任务 | `scheduled_tasks` MCP | crontab + shell 脚本 | crontab + shell 脚本 |
| 文件访问 | `request_cowork_directory` | 直接文件系统 | 直接文件系统 |
| 结果展示 | 交互卡片 | 文件路径 | 文件路径 |

## 文件命名规则

```
工作记录/
├── 2026-05-09-1.md    # 当天第一次记录
├── 2026-05-09-2.md    # 当天第二次记录
├── 2026-05-10-1.md    # 第二天
└── ...
```

- 有对话活动的日期才会生成文件
- 同一天多次记录按序号递增
- 文件名格式：`YYYY-MM-DD-N.md`

## 项目结构

```
daily-work-log/
├── README.md              # 本文件
├── LICENSE                # MIT 许可证
├── .gitignore             # 排除工作记录等隐私文件
├── daily-work-log.skill   # 一键安装包（Cowork）
└── daily-work-log/
    └── SKILL.md           # skill 核心指令
```

## 在多个项目中使用

每个项目独立设置，互不干扰：

1. 在每个项目中分别说"设置自动工作记录"
2. 指定各自的项目路径
3. 系统会为每个项目创建独立的定时任务

## 常见问题

| 问题 | 解答 |
|------|------|
| 定时任务没执行（Cowork） | 首次手动运行一次以授权目录访问 |
| 定时任务没执行（Claude Code） | `crontab -l` 检查 cron 是否注册成功；检查 `claude` CLI 是否在 PATH 中 |
| 记录内容不完整 | 手动模式只总结当前对话；如需完整记录，使用自动定时模式 |
| 想修改执行时间 | 在对应项目中说"设置自动工作记录"重新配置 |
| Windows 没有 crontab | skill 会提供 Task Scheduler 的替代配置指导 |

## 许可证

MIT License — 详见 [LICENSE](./LICENSE)
