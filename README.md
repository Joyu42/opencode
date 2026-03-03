# OpenCode 个人配置指南

## 配置文件总览

```
~/.config/opencode/
├── opencode.json              # 主配置（模型、MCP、插件）
├── oh-my-opencode.json        # Agent / Category 模型分配
├── supermemory.jsonc           # 持久记忆配置
├── instructions/language.md    # 语言指令（中文 + Context7 规则）
├── command/                    # 自定义斜杠命令
├── skills/                     # 15 个技能包
└── README.md                   # ← 本文件
```

---

## 1. 架构概览

### Agent 体系

| 角色 | Agent | 模型 | 用途 |
|------|-------|------|------|
| 指挥官 | Sisyphus | `copilot/claude-opus-4.6` | 主编排器，规划+委派 |
| 战略规划 | Prometheus | `copilot/claude-opus-4.6` | 面试式需求分析 |
| 军师 | Oracle | `copilot/gpt-5.2` | 架构咨询/疑难调试 |
| 侦察兵 | Explore | `minimax/M2.5` | 搜索代码库 |
| 侦察兵 | Librarian | `minimax/M2.5` | 搜索文档/开源 |
| 参谋 | Metis | `minimax/M2.5` | 计划缺口分析 |
| 参谋 | Momus | `minimax/M2.5` | 方案审查验证 |
| 编排 | Atlas | `minimax/M2.5` | Todo 编排执行 |
| 视觉 | Multimodal Looker | `minimax/M2.5` | 截图/图像分析 |

### Category（子任务分发）

| Category | 模型 | 用途 |
|----------|------|------|
| visual-engineering | `copilot/gemini-3.1-pro` | 前端/UI/设计 |
| ultrabrain | `copilot/gemini-3.1-pro` | 高难度逻辑 |
| artistry | `copilot/gemini-3.1-pro` | 创意/非常规方案 |
| quick | `minimax/M2.5` | 简单小改动 |
| unspecified-low | `minimax/M2.5` | 一般任务 |
| unspecified-high | `minimax/M2.5` | 较复杂任务 |
| writing | `minimax/M2.5` | 文档写作 |

### 设计原则

- **Copilot 额度留给核心**：Sisyphus、Oracle、Prometheus
- **MiniMax 承担大量工作**：搜索、检索、辅助分析、轻量执行（快且无限额）
- **特殊任务走 Gemini**：前端/视觉/高难度推理

### 执行流

```
你的输入
   ↓
[Sisyphus] 主编排 (Opus 4.6)
   ├──→ [Explore/Librarian] ×N 并行搜索 (MiniMax) 🔄
   ├──→ [Oracle] 架构咨询 (GPT-5.2) 💰
   ├──→ [Metis/Momus] 计划分析 (MiniMax) 🔄
   ├──→ [Prometheus] 战略规划 (Opus 4.6) 💰
   └──→ [Category 子任务] 按类别分发
         ├── quick / writing → MiniMax 🔄
         └── visual / ultrabrain → Gemini 💰

🔄 = MiniMax（快、无限额）  💰 = Copilot（有限额）
```

---

## 2. 日常使用

### 懒人模式（推荐 90% 场景）

```
ulw 给这个项目加个暗黑模式
```

输入后 Sisyphus 自动探索代码库、搜文档、拆任务、并行执行、验证结果。

### 精确模式（大项目/关键变更）

1. 按 **Tab** → Prometheus 面试式需求分析
2. 生成计划 → `/start-work` → Atlas 按计划执行

### 持续模式

```
/ulw-loop
```

自我循环直到 100% 完成，适合大型重构。

---

## 3. MCP 服务

### 已配置的 MCP

| MCP | 类型 | 用途 |
|-----|------|------|
| **Context7** | remote | 获取最新官方库/框架文档 |
| **MiniMax** | local | 网络搜索 + 图像分析 |

### Context7

按需自动查询最新 API 文档，无需手动触发。

### MiniMax Coding Plan MCP

MiniMax 官方 MCP 服务器，为 Coding Plan 用户提供两个工具：

| 工具 | 功能 | 参数 |
|------|------|------|
| `web_search` | 网络搜索，返回结构化结果 | `query`: 搜索关键词 |
| `understand_image` | AI 图像分析 | `prompt`: 分析指令, `image_source`: 图片路径/URL |

**典型场景**：
- 搜索技术文档、库的使用方法、报错解决方案
- 分析 UI 截图、架构图、流程图
- 从图片中提取文字（OCR）

**支持格式**：JPEG, PNG, WebP

> **注意**：`web_search` 和 `understand_image` 需要有效的 MiniMax API Key 和 Coding Plan 订阅。

---

## 4. 技能（Skills）

| 技能 | 说明 |
|------|------|
| `fetch4ai` | 网页内容抓取（带噪声过滤） |
| `web-research` | 结构化网络调研 |
| `arxiv` | 学术论文检索 |
| `pdf` | PDF 读取/创建/合并/拆分 |
| `docx` | Word 文档处理 |
| `xlsx` | 电子表格处理 |
| `pptx` | PowerPoint 处理 |
| `md-to-docx` | Markdown 转 Word（支持中文排版） |
| `markitdown` | 多格式转 Markdown |
| `marp-slides-creator` | Marp 幻灯片制作 |
| `frontend-design` | 前端界面设计 |
| `cc-insights` | Claude Code 使用分析 |
| `skill-creator` | 自定义技能开发 |
| `command-development` | 斜杠命令开发 |
| `chinese-quote-converter` | 英文引号转中文引号 |

---

## 5. 配置详解

### opencode.json（主配置）

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
      "headers": { "CONTEXT7_API_KEY": "ctx7sk-..." },
      "enabled": true
    },
    "MiniMax": {
      "type": "local",
      "command": ["uvx", "minimax-coding-plan-mcp", "-y"],
      "environment": {
        "MINIMAX_API_KEY": "你的API Key",
        "MINIMAX_API_HOST": "https://api.minimaxi.com"
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

**常用字段**：

| 字段 | 说明 |
|------|------|
| `instructions` | 指令文件路径（支持 glob） |
| `mcp` | MCP 服务器配置 |
| `plugin` | 插件列表 |
| `model` | 默认模型 `provider/model` |
| `autoupdate` | 自动更新 `true`/`false`/`"notify"` |
| `disabled_providers` | 禁用的模型提供商 |

**MCP 类型**：
- `remote`：远程 MCP（配置 `url` + 可选 `headers`）
- `local`：本地进程（配置 `command` + 可选 `environment`）

**变量替换**：`{env:VAR_NAME}` 引用环境变量，`{file:~/.secrets/key}` 引用文件内容

### oh-my-opencode.json（Agent 配置）

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/dev/assets/oh-my-opencode.schema.json",
  "disabled_hooks": ["anthropic-context-window-limit-recovery"],
  "agents": {
    "sisyphus": { "model": "github-copilot/claude-opus-4.6", "variant": "max" },
    "oracle": { "model": "github-copilot/gpt-5.2", "variant": "high" },
    "explore": { "model": "minimax-cn/MiniMax-M2.5" },
    "librarian": { "model": "minimax-cn/MiniMax-M2.5" }
  },
  "categories": {
    "visual-engineering": { "model": "github-copilot/gemini-3.1-pro-preview", "variant": "high" },
    "quick": { "model": "minimax-cn/MiniMax-M2.5" }
  }
}
```

**Agent 配置项**：

| 选项 | 说明 |
|------|------|
| `model` | 模型覆盖 `provider/model` |
| `variant` | 变体: `max`, `xhigh`, `high`, `medium`, `low` |
| `fallback_models` | API 失败时备用模型 |
| `temperature` | 采样温度 (0-2) |
| `prompt_append` | 追加系统提示词 |
| `tools` | 工具白名单 `{ "toolname": true/false }` |
| `disable` | 禁用此 Agent |

**可禁用的 Hooks**（完整列表见 `disabled_hooks`）：

| Hook | 说明 |
|------|------|
| `anthropic-context-window-limit-recovery` | Anthropic 上下文恢复（Supermemory 需禁用） |
| `todo-continuation-enforcer` | Todo 继续强制器 |
| `context-window-monitor` | 上下文窗口监控 |
| `ralph-loop` | Ralph 循环 |
| `think-mode` | 思考模式 |
| `runtime-fallback` | 运行时回退 |

### supermemory.jsonc（持久记忆）

```jsonc
{
  "apiKey": "sm_...",
  "similarityThreshold": 0.6,   // 检索最小相似度
  "maxMemories": 5,             // 每次注入的最大记忆数
  "compactionThreshold": 0.80   // 触发压缩的上下文使用率
}
```

从 [console.supermemory.ai](https://console.supermemory.ai) 获取 API Key。

---

## 6. 注意事项

### MiniMax 表现良好的场景

- **Explore / Librarian**：最佳位置，快且可大量并行
- **quick / writing**：轻量任务绰绰有余
- **Atlas**：Todo 编排相对简单

### 可能需要关注的场景

- **Metis / Momus**：深度分析偶尔不如 GPT，但大部分够用
- **unspecified-high**：复杂任务质量不够可改回 `copilot/claude-sonnet-4.5`

### 不可更改

- **Sisyphus 必须用 Claude 家族**，换模型会严重退化
- **Hephaestus 必须用 GPT-5.3 Codex**，专为 Codex 构建

### 省额度技巧

- 简单任务用 `ulw`，不走 Prometheus
- 避免反复让 Oracle 解答同类问题

---

## 7. 相关链接

- [OpenCode 官方文档](https://opencode.ai/docs/config/)
- [OhMyOpenCode GitHub](https://github.com/code-yeongyu/oh-my-opencode)
- [MiniMax Coding Plan MCP](https://github.com/MiniMax-AI/MiniMax-Coding-Plan-MCP)
- [MiniMax 平台](https://platform.minimaxi.com)
- [Supermemory Console](https://console.supermemory.ai)
- [Context7 MCP](https://mcp.context7.com)

---

*更新日期：2026-03-04*
