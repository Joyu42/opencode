# OpenCode 配置文件完全指南

本文档详细说明 `~/.config/opencode/` 目录下所有配置文件的用途、字段说明和配置方法。

---

## 目录

1. [配置文件概览](#1-配置文件概览)
2. [opencode.json - 主配置](#2-opencodejson---主配置)
3. [oh-my-opencode.json - Agent 配置](#3-oh-my-opencodejson---agent-配置)
4. [supermemory.jsonc - 记忆配置](#4-supermemoryjsonc---记忆配置)
5. [instructions - 指令文件](#5-instructions---指令文件)
6. [skills - 技能配置](#6-skills---技能配置)
7. [command - 自定义命令](#7-command---自定义命令)
8. [常用配置示例](#8-常用配置示例)

---

## 1. 配置文件概览

```
~/.config/opencode/
├── opencode.json              # OpenCode 主配置文件
├── oh-my-opencode.json       # OhMyOpenCode Agent 配置
├── supermemory.jsonc         # Supermemory 记忆配置
├── instructions/
│   └── language.md            # 语言设置指令
├── command/
│   ├── supermemory-init.md   # Supermemory 初始化命令
│   └── supermemory-login.md  # Supermemory 登录命令
├── skills/                    # 技能目录 (15个技能)
├── package.json              # 插件依赖
└── README.md                 # 配置文件说明
```

---

## 2. opencode.json - 主配置

### 文件位置

| 优先级 | 位置 | 说明 |
|--------|------|------|
| 1 | 远程 `.well-known/opencode` | 组织默认配置 |
| 2 | `~/.config/opencode/opencode.json` | 全局用户配置 |
| 3 | `OPENCODE_CONFIG` 环境变量 | 自定义路径 |
| 4 | `./opencode.json` | 项目根目录 |
| 5 | `./.opencode/opencode.json` | 项目 Agent/命令/插件 |

> **注意**: 配置是**合并**而非替换。冲突时后面的优先级更高。

### 当前配置

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "~/.config/opencode/instructions/language.md"
  ],
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "ctx7sk-f9c168b1-3d9e-45b7-9bc5-03876801340b"
      },
      "enabled": true
    }
  },
  "plugin": [
    "oh-my-opencode@latest",
    "opencode-supermemory@latest"
  ]
}
```

### 完整字段参考

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `$schema` | `string` | - | JSON Schema 校验路径 |
| `model` | `string` | - | 默认使用的模型，格式: `provider/model` |
| `small_model` | `string` | - | 轻量任务专用模型（如标题生成） |
| `default_agent` | `string` | `build` | 默认 Agent（build/plan/自定义） |
| `instructions` | `array` | - | 指令文件路径数组（支持 glob） |
| `mcp` | `object` | - | MCP 服务器配置 |
| `plugin` | `array` | - | 加载的插件列表（npm 包名） |
| `server` | `object` | - | 服务器配置（端口、主机名等） |
| `tui` | `object` | - | 终端 UI 主题配置 |
| `autoupdate` | `boolean \| string` | `true` | 自动更新: `true`/`false`/`"notify"` |
| `share` | `string` | `manual` | 分享行为: `manual`/`auto`/`disabled` |
| `disabled_providers` | `array` | - | 禁用的模型提供商 |
| `enabled_providers` | `array` | - | 仅启用的模型提供商 |
| `command` | `object` | - | 自定义命令配置 |
| `formatter` | `object` | - | 代码格式化器配置 |
| `compaction` | `object` | - | 上下文压缩配置 |
| `watcher` | `object` | - | 文件监视器忽略模式 |
| `contextPaths` | `array` | - | 上下文文件路径 |

### MCP 配置示例

```json
"mcp": {
  "context7": {
    "type": "remote",
    "url": "https://mcp.context7.com/mcp",
    "headers": {
      "CONTEXT7_API_KEY": "ctx7sk-..."
    },
    "enabled": true
  },
  "jira": {
    "type": "remote",
    "url": "https://jira.example.com/mcp",
    "enabled": true
  }
}
```

### 变量替换

```json
// 环境变量
"apiKey": "{env:ANTHROPIC_API_KEY}"

// 文件内容
"apiKey": "{file:~/.secrets/key}"
```

---

## 3. oh-my-opencode.json - Agent 配置

### 文件位置

- `~/.config/opencode/oh-my-opencode.json`
- `.opencode/oh-my-opencode.json` (项目级)

启用自动补全:
```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/dev/assets/oh-my-opencode.schema.json"
}
```

### 当前配置

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/dev/assets/oh-my-opencode.schema.json",
  "disabled_hooks": ["anthropic-context-window-limit-recovery"],
  "agents": {
    "sisyphus": {
      "model": "github-copilot/claude-opus-4.6",
      "variant": "max"
    },
    "oracle": {
      "model": "github-copilot/gpt-5.2",
      "variant": "high"
    },
    "librarian": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "explore": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "multimodal-looker": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "prometheus": {
      "model": "github-copilot/claude-opus-4.6",
      "variant": "max"
    },
    "metis": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "momus": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "atlas": {
      "model": "minimax-cn/MiniMax-M2.5"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "variant": "high"
    },
    "ultrabrain": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "variant": "high"
    },
    "artistry": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "variant": "high"
    },
    "quick": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "unspecified-low": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "unspecified-high": {
      "model": "minimax-cn/MiniMax-M2.5"
    },
    "writing": {
      "model": "minimax-cn/MiniMax-M2.5"
    }
  }
}
```

### 顶层配置字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `$schema` | `string` | Schema URL 用于自动补全 |
| `new_task_system_enabled` | `boolean` | 启用新任务系统 |
| `default_run_agent` | `string` | 默认运行的 Agent |
| `disabled_mcps` | `array` | 禁用的 MCP: `websearch`, `context7`, `grep_app` |
| `disabled_agents` | `array` | 禁用的 Agent |
| `disabled_skills` | `array` | 禁用的 Skills |
| `disabled_hooks` | `array` | 禁用的 Hooks |
| `disabled_commands` | `array` | 禁用的命令 |
| `disabled_tools` | `array` | 禁用的工具 |
| `hashline_edit` | `boolean` | 启用哈希锚点编辑 |
| `model_fallback` | `boolean` | 启用运行时回退 |

### Agent 配置

**可用 Agents**: `sisyphus`, `hephaestus`, `prometheus`, `oracle`, `librarian`, `explore`, `multimodal-looker`, `metis`, `momus`, `atlas`

| 选项 | 类型 | 说明 |
|------|------|------|
| `model` | `string` | 模型覆盖，格式: `provider/model` |
| `variant` | `string` | 模型变体: `max`, `high`, `medium`, `low`, `xhigh` |
| `fallback_models` | `string \| array` | API 错误时的备用模型 |
| `temperature` | `number` | 采样温度 (0-2) |
| `top_p` | `number` | Top-p 采样 (0-1) |
| `prompt` | `string` | 替换系统提示词 |
| `prompt_append` | `string` | 追加到系统提示词 |
| `tools` | `object` | 允许的工具: `{ "toolname": true/false }` |
| `disable` | `boolean` | 禁用此 Agent |
| `description` | `string` | Agent 描述 |
| `mode` | `string` | Agent 模式: `subagent`, `primary`, `all` |
| `color` | `string` | UI 颜色 (十六进制如 `#FF0000`) |
| `maxTokens` | `number` | 最大响应 tokens |
| `thinking` | `object` | Anthropic 扩展思考: `{ "type": "enabled"/"disabled", "budgetTokens": number }` |
| `reasoningEffort` | `string` | OpenAI 推理: `low`, `medium`, `high`, `xhigh` |
| `textVerbosity` | `string` | 文本详细度: `low`, `medium`, `high` |
| `permission` | `object` | 逐工具权限 |

### Category 配置

**内置 Categories**: `visual-engineering`, `ultrabrain`, `deep`, `artistry`, `quick`, `unspecified-low`, `unspecified-high`, `writing`

| 选项 | 类型 | 说明 |
|------|------|------|
| `model` | `string` | 模型覆盖 |
| `variant` | `string` | 模型变体 |
| `temperature` | `number` | 采样温度 |
| `top_p` | `number` | Top-p 采样 |
| `maxTokens` | `number` | 最大 tokens |
| `thinking` | `object` | 扩展思考配置 |
| `tools` | `array` | 允许的工具列表 |
| `prompt_append` | `string` | 追加提示词 |
| `description` | `string` | 显示在 task() 工具中 |
| `is_unstable_agent` | `boolean` | 强制后台模式 |

### Hooks (禁用钩子)

可禁用的 Hooks:

| Hook | 说明 |
|------|------|
| `todo-continuation-enforcer` | Todo 继续强制器 |
| `context-window-monitor` | 上下文窗口监控 |
| `session-recovery` | 会话恢复 |
| `session-notification` | 会话通知 |
| `comment-checker` | 注释检查 |
| `grep-output-truncator` | Grep 输出截断 |
| `tool-output-truncator` | 工具输出截断 |
| `directory-agents-injector` | 目录 Agent 注入 |
| `directory-readme-injector` | 目录 README 注入 |
| `empty-task-response-detector` | 空任务响应检测 |
| `think-mode` | 思考模式 |
| `anthropic-context-window-limit-recovery` | **Anthropic 上下文恢复**（Supermemory 需禁用） |
| `rules-injector` | 规则注入 |
| `background-notification` | 后台通知 |
| `auto-update-checker` | 自动更新检查 |
| `startup-toast` | 启动提示 |
| `keyword-detector` | 关键词检测 |
| `agent-usage-reminder` | Agent 使用提醒 |
| `non-interactive-env` | 非交互环境 |
| `interactive-bash-session` | 交互式 Bash 会话 |
| `compaction-context-injector` | 压缩上下文注入 |
| `thinking-block-validator` | 思考块验证 |
| `claude-code-hooks` | Claude Code 钩子 |
| `ralph-loop` | Ralph 循环 |
| `preemptive-compaction` | 抢先压缩 |
| `auto-slash-command` | 自动斜杠命令 |
| `start-work` | 开始工作 |
| `runtime-fallback` | 运行时回退 |

---

## 4. supermemory.jsonc - 记忆配置

### 文件位置

`~/.config/opencode/supermemory.jsonc`

### 当前配置

```jsonc
{
  "apiKey": "sm_3Ua6TpeE62ujg3eCCGNBpF_IPYwkMHChbJSvrxtvBHxTcMasZlKCIRSZAdfzMENABChhKElPhuSkBJvgBmNikMC"
}
```

### 完整字段参考

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `apiKey` | `string` | 环境变量 `SUPERMEMORY_API_KEY` | 从 [console.supermemory.ai](https://console.supermemory.ai) 获取 |
| `similarityThreshold` | `number` (0-1) | `0.6` | 记忆检索最小相似度分数 |
| `maxMemories` | `number` | `5` | 每次请求注入的最大记忆数量 |
| `maxProjectMemories` | `number` | `10` | 列出最大项目记忆数量 |
| `maxProfileItems` | `number` | `5` | 注入的最大用户资料项数量 |
| `injectProfile` | `boolean` | `true` | 是否包含用户资料 |
| `containerTagPrefix` | `string` | `"opencode"` | 自动生成容器标签的前缀 |
| `userContainerTag` | `string` | 自动生成 | 覆盖用户容器标签 |
| `projectContainerTag` | `string` | 自动生成 | 覆盖项目容器标签 |
| `keywordPatterns` | `string[]` | 内置默认 | 触发记忆检测的额外正则模式 |
| `compactionThreshold` | `number` (0-1) | `0.80` | 触发抢先压缩的上下文使用率 |

### 默认关键词模式（内置，无法禁用）

```
remember, memorize, save this, note this, keep in mind, don't forget, 
learn this, store this, record this, make a note, take note, jot down, 
commit to memory, remember that, never forget, always remember
```

### 完整配置示例

```jsonc
{
  "apiKey": "sm_...",

  // 记忆检索设置
  "similarityThreshold": 0.6,
  "maxMemories": 5,
  "maxProjectMemories": 10,
  "maxProfileItems": 5,
  "injectProfile": true,

  // 容器标签设置
  "containerTagPrefix": "opencode",
  
  // 自定义精确标签
  "userContainerTag": "my-team-workspace",
  "projectContainerTag": "my-awesome-project",

  // 记忆检测
  "keywordPatterns": [
    "log\\s+this",
    "write\\s+down",
    "capture\\this"
  ],

  // 压缩行为
  "compactionThreshold": 0.80
}
```

### 优先级顺序

**API Key**: `SUPERMEMORY_API_KEY` 环境变量 → 配置文件 `apiKey` → OAuth 凭证

---

## 5. instructions - 指令文件

### language.md

```markdown
# 语言设置

你是一个AI编程助手。请始终使用简体中文进行回复，包括思考过程和最终回答。

在以下情况下使用简体中文：
- 思考过程和分析
- 代码注释和解释
- 回复用户的任何问题
- 提供技术文档和说明

除非用户明确要求使用其他语言，否则请始终使用简体中文。

---

## Context7 MCP 使用规则

当我需要库/API文档、代码生成、设置或配置步骤时，必须主动使用 Context7 MCP 工具，无需用户明确要求。
```

---

## 6. skills - 技能配置

### 技能列表

| 技能名称 | 说明 |
|---------|------|
| `arxiv` | 学术论文检索 |
| `cc-insights` | 代码分析洞察 |
| `chinese-quote-converter` | 中文引号转换 |
| `command-development` | 命令行工具开发 |
| `docx` | Word 文档处理 |
| `fetch4ai` | 网页内容抓取 |
| `frontend-design` | 前端界面设计 |
| `markitdown` | Markdown 格式转换 |
| `marp-slides-creator` | 幻灯片制作 |
| `md-to-docx` | Markdown 转 Word |
| `pdf` | PDF 文件处理 |
| `pptx` | PowerPoint 处理 |
| `skill-creator` | 自定义技能开发 |
| `web-research` | 网络信息搜集 |
| `xlsx` | 电子表格处理 |

### 技能文件结构

每个技能目录包含:
- `SKILL.md` - 技能定义和使用说明
- `references/` - 参考文档
- `scripts/` - 脚本文件
- `themes/` - 主题文件（如适用）

---

## 7. command - 自定义命令

### supermemory-init.md

```markdown
---
description: Initialize Supermemory with comprehensive codebase knowledge
---

# Initializing Supermemory

[详细的 Supermemory 初始化指南...]
```

### supermemory-login.md

```markdown
---
description: Authenticate with Supermemory via browser
---

# Supermemory Login

Run this command to authenticate the user with Supermemory:

bash
bunx opencode-supermemory@latest login


```

---

## 8. 常用配置示例

### 8.1 基础配置（仅 OpenCode）

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "~/.config/opencode/instructions/language.md"
  ],
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "ctx7sk-..."
      },
      "enabled": true
    }
  },
  "plugin": []
}
```

### 8.2 完整配置（OhMyOpenCode + Supermemory）

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "~/.config/opencode/instructions/language.md"
  ],
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "ctx7sk-..."
      },
      "enabled": true
    }
  },
  "plugin": [
    "oh-my-opencode@latest",
    "opencode-supermemory@latest"
  ]
}
```

```jsonc
// oh-my-opencode.json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/dev/assets/oh-my-opencode.schema.json",
  "disabled_hooks": ["anthropic-context-window-limit-recovery"],
  "agents": {
    "sisyphus": {
      "model": "github-copilot/claude-opus-4.6",
      "variant": "max"
    },
    "oracle": {
      "model": "github-copilot/gpt-5.2",
      "variant": "high"
    }
  }
}
```

```jsonc
// supermemory.jsonc
{
  "apiKey": "sm_...",
  "similarityThreshold": 0.6,
  "maxMemories": 5,
  "compactionThreshold": 0.80
}
```

---

## 相关链接

- [OpenCode 官方文档](https://opencode.ai/docs/config/)
- [OpenCode JSON Schema](https://opencode.ai/config.json)
- [OhMyOpenCode GitHub](https://github.com/code-yeongyu/oh-my-opencode)
- [Supermemory GitHub](https://github.com/supermemoryai/opencode-supermemory)
- [Supermemory Console](https://console.supermemory.ai)

---

*文档生成日期: 2026-03-03*
