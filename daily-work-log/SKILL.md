---
name: daily-work-log
description: "Automatically record daily work progress for any project. Works on Claude desktop (Cowork), Claude Code CLI, and IDE plugins. Generates structured Markdown files summarizing conversations, code changes, and results. Use this skill whenever the user wants to set up automatic daily work logging, record today's work progress, generate a daily work summary, create a work journal, or track what they did each day. Trigger when user mentions work logging, work recording, daily summary, work journal, daily report, or wants to track daily coding activity."
---

# Daily Work Log

A cross-platform skill for automatically recording daily project work into structured Markdown files. Supports **Cowork (Claude desktop)**, **Claude Code (CLI)**, and **IDE plugins** (VS Code, JetBrains).

## Platform detection

Before executing any mode, determine which platform you are running on:

- **Cowork**: You have access to `mcp__session_info__*`, `mcp__scheduled-tasks__*`, `mcp__cowork__*` tools
- **Claude Code**: You are running in a terminal, have direct filesystem access via bash, can use `crontab` or `claude` CLI
- **IDE plugin**: You are embedded in VS Code or JetBrains, have direct filesystem access

Follow the platform-specific instructions in each section. If unsure which platform, ask the user.

## When to use this skill

This skill has two modes:

- **Setup mode**: User says "set up automatic work logging", "auto-record my daily work", "configure daily work reports", etc. Creates a scheduled task (platform-specific) that runs daily.
- **Manual mode**: User says "record today's work", "summarize what I did today", "generate yesterday's work report", etc. Immediately produces a work record.

If ambiguous, ask the user.

---

## Setup mode (all platforms)

### Step 1: Gather configuration

Ask the user (use `AskUserQuestion` in Cowork, inline questions otherwise):

1. **Project directory path**: Absolute path to the project folder
2. **Schedule time**: When to run daily? Default 6:00 AM. Use cron format in LOCAL time.

### Step 2: Confirm before creating

Present a summary and ask for final confirmation:

- Project path
- Recordings path: `<path>/工作记录/`
- Schedule: described in plain language
- File naming: `YYYY-MM-DD-N.md`
- Behavior: only creates records on days with activity

### Step 3: Create work-records folder

```bash
mkdir -p "<project_path>/工作记录"
```

### Step 4: Create the scheduled task (platform-specific)

#### Cowork

Use the `create_scheduled_task` tool:
- **taskId**: `daily-work-log-<project-name>`
- **description**: "Daily work log for <project-name>"
- **cronExpression**: From user's input (default `0 6 * * *`)
- **prompt**: Use the **Cowork Scheduled Task Prompt** below

#### Claude Code / IDE plugin

Create a shell script at `<project_path>/.daily-work-log.sh`:

```bash
#!/bin/bash
# Daily Work Log - auto-generated, do not edit manually
PROJECT_PATH="<project_path>"
DATE=$(date -d "yesterday" +%Y-%m-%d)
OUTPUT_DIR="$PROJECT_PATH/工作记录"
mkdir -p "$OUTPUT_DIR"

# Find next sequence number
N=1
while [ -f "$OUTPUT_DIR/${DATE}-${N}.md" ]; do
  N=$((N + 1))
done

# Run Claude to generate the work record
claude -p "$(cat << 'CLAUDE_PROMPT'
You are generating a daily work record. Summarize all work done yesterday ($DATE) for the project at $PROJECT_PATH.

Check the conversation history files in ~/.claude/ or the project's .claude/ directory for sessions from $DATE. 

IMPORTANT: Only include sessions related to the project at $PROJECT_PATH. A session belongs to this project if it mentions the project path or discusses files within it. Skip sessions from other projects.

If there are NO sessions about this project from that date, respond with "NO_ACTIVITY" and nothing else.

If there were relevant sessions, generate a Markdown work record following this exact template:

# 工作记录 - $(date -d "$DATE" +%Y年%m月%d日)

## 一、工作概述
[2-4 sentence summary]

## 二、工作内容与进展
[Flowing narrative summarizing all work — what was done, code written, results achieved, decisions made. Synthesize into one coherent narrative, do NOT list conversations individually.]

## 三、文件变更汇总
- **新建**: [list or 无]
- **修改**: [list or 无]

## 四、问题与笔记
- [issues or 无]

---
*自动生成于 $(date +%Y-%m-%d %H:%M)*

Output ONLY the Markdown content. Save it yourself is not needed — the calling script handles that.
CLAUDE_PROMPT
)" > "$OUTPUT_DIR/${DATE}-${N}.md" 2>/dev/null

# Check if the file has content (not just "NO_ACTIVITY")
if grep -q "NO_ACTIVITY" "$OUTPUT_DIR/${DATE}-${N}.md" 2>/dev/null; then
  rm "$OUTPUT_DIR/${DATE}-${N}.md"
fi
```

Make it executable:
```bash
chmod +x "<project_path>/.daily-work-log.sh"
```

Then register a cron job. Show the user the command and ask them to run it in their terminal:

```bash
(crontab -l 2>/dev/null; echo "0 6 * * * <project_path>/.daily-work-log.sh") | crontab -
```

Tell the user:
- The cron job runs daily at the configured time
- They can verify with `crontab -l`
- To remove: `crontab -e` and delete the line

### Step 5: Confirm setup

Tell the user the setup is complete and explain where records will be saved.

---

## Manual mode (all platforms)

The manual mode works the same across all platforms — summarize the CURRENT conversation context. No need to read external session files.

### Step 1: Determine the date

- "today" → today's date
- "yesterday" or morning → yesterday's date
- Specific date → that date
- Default: today

### Step 2: Summarize from current conversation

You already have the full conversation context. Scan the current session for:
- What tasks were discussed and accomplished
- What code was written or modified
- What results were achieved
- Any decisions made
- Problems or TODOs

Synthesize into one coherent narrative — do NOT list conversations individually.

### Step 3: Determine file sequence number

```bash
ls "<project_path>/工作记录/<date>-"*.md 2>/dev/null
```

Count existing files. Use next number: `<date>-1.md`, `<date>-2.md`, etc.

### Step 4: Write the Markdown record

Use the `Write` tool (or bash `cat` in Claude Code) to save `<project_path>/工作记录/<date>-<N>.md`.

### Step 5: Present the result

- **Cowork**: Use `present_files`
- **Claude Code / IDE**: Show the absolute file path to the user

---

## Cowork Scheduled Task Prompt

When creating a scheduled task in Cowork, use this self-contained prompt:

```
You are executing a daily work log task for the project at {PROJECT_PATH}.

## Objective
Generate a structured Markdown file summarizing all work done on the PREVIOUS calendar day. Save to {PROJECT_PATH}/工作记录/YYYY-MM-DD-N.md. If no activity, do nothing.

## Steps

### 1. Calculate target date
YESTERDAY. Format: YYYY-MM-DD.

### 2. Ensure project access
Use request_cowork_directory with path={PROJECT_PATH} if needed.

### 3. Read and filter session transcripts

IMPORTANT: You are recording work for ONE specific project at {PROJECT_PATH}. Do NOT include sessions from other projects.

Use mcp__session_info__list_sessions to get recent sessions. For each session that may be from the target date:
1. Read the transcript with mcp__session_info__read_transcript
2. Check if the session is about THIS project. A session belongs to this project if:
   - The transcript mentions the project path {PROJECT_PATH}
   - Files discussed are within the project directory
   - The conversation topic is clearly about this project's work
3. Skip sessions that are about other projects or unrelated topics

After filtering, extract from relevant sessions only:
- Tasks discussed and accomplished
- Code written or modified (file paths, key functions)
- Results achieved
- Decisions made
- Problems or TODOs

Synthesize into one coherent narrative.

### 4. Skip if no activity
If no sessions found: report "No activity" and stop.

### 5. Determine file sequence number
ls "{PROJECT_PATH}/工作记录/<date>-"*.md 2>/dev/null
Use next number: <date>-1.md, <date>-2.md, etc.

### 6. Create the work record
Use Write tool. Follow this template:

# 工作记录 - YYYY年MM月DD日

## 一、工作概述
[2-4 sentence summary]

## 二、工作内容与进展
[Flowing narrative — what was done, code written, results, decisions. Synthesize, do NOT list conversations individually.]

## 三、文件变更汇总
- **新建**: [list or 无]
- **修改**: [list or 无]

## 四、问题与笔记
- [issues or 无]

---
*自动生成于 YYYY-MM-DD HH:MM*

### 7. Save
Save to {PROJECT_PATH}/工作记录/<date>-<N>.md. Create folder if needed. Use present_files.

### 8. Report
Report: "Daily work record saved: {PROJECT_PATH}/工作记录/<date>-<N>.md"
```

---

## Document Template

```markdown
# 工作记录 - YYYY年MM月DD日

## 一、工作概述

[2-4 sentence high-level summary]

## 二、工作内容与进展

[Flowing narrative — tasks, code written, results, decisions. Synthesize into one coherent narrative, do NOT list conversations individually.]

## 三、文件变更汇总

- **新建**: file1.py, file2.js (or 无)
- **修改**: file3.md, file4.py (or 无)

## 四、问题与笔记

- [Issue or TODO]
(or 无)

---

*自动生成于 YYYY-MM-DD HH:MM*
```

### Template rules

- Title: `#` (H1), sections: `##` (H2)
- Section 二 is a flowing narrative, NOT per-conversation
- File paths relative to project root when possible
- Footer: `*自动生成于 YYYY-MM-DD HH:MM*`

---

## Platform-specific notes

### Cowork
- Session reading: `mcp__session_info__*` tools
- Scheduling: `create_scheduled_task`
- File access: `request_cowork_directory` if needed
- Present: `present_files`

### Claude Code
- Session reading (manual mode): current conversation context
- Session reading (scheduled task): check `~/.claude/` history files
- Scheduling: crontab + shell script
- File access: direct bash
- Present: show file path

### IDE plugin
- Session reading (manual mode): current conversation context
- Session reading (scheduled task): check IDE's Claude history directory
- Scheduling: same as Claude Code (crontab + script)
- File access: direct filesystem
- Present: show file path

## Important notes

- **Skip empty days**: Never create records on days with no activity
- **Multi-session days**: Synthesize all sessions into one narrative in section 二
- **No dependencies**: This skill needs no Python packages or third-party tools
- **cron prerequisite** (Claude Code/IDE): The user's system must have `crontab` available (standard on Linux/macOS; Windows users can use Task Scheduler — provide equivalent instructions if needed)
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          