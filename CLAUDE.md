# 自动保存记录 Skill 项目

## 项目概述

研发一个名为 `daily-work-log` 的 Claude skill，用于自动记录每日项目工作内容。支持 Cowork（桌面）、Claude Code（终端）、IDE 插件三个平台。

## Skill 核心信息

- **名称**: daily-work-log
- **仓库**: https://github.com/bonjourGYF/daily-work-log
- **已安装**: 是（通过 save_skill 注册到系统）
- **定时任务**: daily-work-log-auto-record-skill（每天早上 6:10 执行）

## 功能要点

1. **Setup 模式**: 引导用户配置项目路径和执行时间 → 询问确认 → 创建定时任务
2. **Manual 模式**: 直接总结当前对话内容，生成 Markdown 工作记录
3. **项目隔离**: 按项目路径过滤会话，不同项目互不干扰
4. **文件命名**: `YYYY-MM-DD-N.md`，同一天多次记录编号递增
5. **零依赖**: 不需要任何 Python 包或第三方工具

## 关键设计决策（已确认）

- 输出格式: Markdown（不是 Word）
- 不包含 git 提交记录（用户不需要）
- 文档模板: 四个章节（概述 / 工作内容与进展 / 文件变更 / 问题笔记）
- 工作内容为合成式叙述，不是逐条对话记录
- 平台差异: Cowork 用 MCP 工具 / Claude Code 用 crontab + shell
- 已修复: 会话读取必须按项目路径过滤（之前把全部项目的对话都记录了）

## 文件结构

```
自动保存记录skill/
├── CLAUDE.md             # 本文件（项目记忆）
├── README.md             # GitHub 说明文档
├── LICENSE               # MIT
├── .gitignore            # 排除 工作记录/ 等
├── daily-work-log.skill  # 安装包
├── daily-work-log/
│   └── SKILL.md          # skill 核心指令
└── 工作记录/             # 生成的日志（不入 git）
```

## 当前状态

- skill 已完成开发、安装、本地测试通过
- 定时任务已创建，等待明天早上首次自动运行验证
- 已上传 GitHub
- Manual 模式测试通过（生成了 2026-05-09-1.md 和 -2.md）

## 待验证 / 待办

- 明天早上定时任务首次自动运行是否正常
- Claude Code 平台的 cron 方案尚未实测
- 后续可能需要: 多语言支持、PDF 输出、邮件通知等扩展功能
