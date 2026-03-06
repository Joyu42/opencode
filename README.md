# OpenCode 个人配置指南

## 插件体系概览

```
~/.config/opencode/
├── opencode.json              # 主配置（模型、MCP、插件）
├── oh-my-opencode.json        # Agent / Category 模型分配（OhMyOpenCode）
├── supermemory.jsonc          # 持久记忆配置（Supermemory）
├── instructions/              # 语言指令
├── command/                   # 自定义斜杠命令
├── skills/                    # 技能包
│   ├── superpowers/          # Superpowers 技能（14个）
│   └── [其他技能]            # 用户技能
└── README.md                  # ← 本文件
```

---

## 三大插件职责分工

| 插件 | 职责 | 层面 | 触发方式 |
|------|------|------|----------|
| **OhMyOpenCode** | 任务编排、Agent调度、规划执行分离 | 架构层 | `@plan` / `/start-work` / `ulw` |
| **Superpowers** | 开发纪律、TDD、代码审查、调试方法论 | 执行层 | 自动检测 + 手动加载 |
| **Supermemory** | 长期记忆、学习代码风格、记住项目知识 | 知识层 | 自动运行 |

### 协作流程

```
用户需求
    ↓
┌─────────────────────────────────────────────┐
│  OhMyOpenCode (编排层)                      │
│  @plan → Prometheus 规划 → /start-work     │
└─────────────────────────────────────────────┘
    ↓ 产出: 实施计划
    ↓
┌─────────────────────────────────────────────┐
│  Superpowers (纪律层)                       │
│  自动注入: brainstorming → TDD → 验证      │
└─────────────────────────────────────────────┘
    ↓ 执行具体任务
    ↓
┌─────────────────────────────────────────────┐
│  Supermemory (知识层)                        │
│  自动记住: 对话、偏好、项目知识              │
└─────────────────────────────────────────────┘
```

---

## 1. OhMyOpenCode - 任务编排

### Agent 体系

| 角色 | Agent | 模型 | 用途 |
|------|-------|------|------|
| 指挥官 | Sisyphus | `copilot/gpt-5.2` | 主编排器，规划+委派 |
| 战略规划 | Prometheus | `copilot/gpt-5.2` | 面试式需求分析 |
| 军师 | Oracle | `copilot/gpt-5.2` | 架构咨询/疑难调试 |
| 侦察兵 | Explore | `minimax/M2.5` | 搜索代码库 |
| 侦察兵 | Librarian | `minimax/M2.5` | 搜索文档/开源 |
| 参谋 | Metis | `minimax/M2.5` | 计划缺口分析 |
| 参谋 | Momus | `minimax/M2.5` | 方案审查验证 |
| 编排 | Atlas | `minimax/M2.5` | Todo 编排执行 |

### Category（子任务分发）

| Category | 模型 | 用途 |
|----------|------|------|
| visual-engineering | `gemini-3.1-pro` | 前端/UI/设计 |
| ultrabrain | `gemini-3.1-pro` | 高难度逻辑 |
| artistry | `gemini-3.1-pro` | 创意/非常规方案 |
| quick | `minimax/M2.5` | 简单小改动 |
| unspecified-low | `minimax/M2.5` | 一般任务 |
| unspecified-high | `minimax/M2.5` | 较复杂任务 |
| writing | `minimax/M2.5` | 文档写作 |

### 使用方式

| 场景 | 命令 | 说明 |
|------|------|------|
| **懒人模式（Sisyphus）** | `ulw 添加暗黑模式` | 自动探索+执行 |
| **精确规划（Prometheus）** | @plan→系列问答 → `/start-work` | Prometheus 规划 + Atlas 执行 |
| **持续模式** | `/ulw-loop` | 自我循环直到完成 |
| **深度架构** | 切换到 Hephaestus | GPT-5.3 Codex 推理 |

---

## 2. Superpowers - 开发纪律

> 安装位置: `~/.config/opencode/superpowers`
> 技能位置: `~/.config/opencode/skills/superpowers`

### 14 个技能

| 类别 | 技能 | 说明 |
|------|------|------|
| **测试** | test-driven-development | TDD 先写测试再写代码 |
| **调试** | systematic-debugging | 系统化调试方法 |
| **调试** | verification-before-completion | 完成后必须验证 |
| **计划** | writing-plans | 编写实施计划 |
| **计划** | executing-plans | 执行计划 |
| **协作** | brainstorming | 创造性工作前必须头脑风暴 |
| **协作** | requesting-code-review | 请求代码审查 |
| **协作** | receiving-code-review | 接收代码审查反馈 |
| **协作** | subagent-driven-development | 子代理驱动开发 |
| **协作** | dispatching-parallel-agents | 并行代理分发 |
| **Git** | using-git-worktrees | Git worktree 隔离开发 |
| **Git** | finishing-a-development-branch | 完成开发分支 |
| **元** | using-superpowers | 使用 superpowers 本身 |
| **元** | writing-skills | 编写新技能 |

### 使用方式

**自动触发**: Superpowers 会自动检测任务类型并注入相关技能。

**手动加载**:
```
use skill tool to load superpowers/brainstorming
use skill tool to load superpowers/test-driven-development
```

### 核心原则

Superpowers 强制执行 **先思考，后行动**：
1. 创造性工作前 → 必须 brainstorming
2. 实现功能前 → 必须 TDD
3. 完成任务前 → 必须 verification-before-completion
4. 代码提交前 → 应该 requesting-code-review

---

## 3. Supermemory - 长期记忆

> 配置文件: `~/.config/opencode/supermemory.jsonc`
> 需禁用 Hook: `anthropic-context-window-limit-recovery`

### 功能

- ✅ 自动记住对话内容
- ✅ 学习代码风格偏好
- ✅ 记录项目配置和架构
- ✅ 跨会话记忆

### 使用方式

**完全自动运行**，无需手动触发。

**手动添加记忆**:
```bash
# 添加项目级别记忆
supermemory add "这个项目使用 PostgreSQL，不是 MySQL" --scope project --type project-config

# 添加用户级别记忆（跨项目）
supermemory add "我喜欢使用 TypeScript strict 模式" --scope user --type preference

# 搜索记忆
supermemory search "数据库"
```

### 配置 (supermemory.jsonc)

```jsonc
{
  "apiKey": "sm_你的密钥",
  "similarityThreshold": 0.6,    // 检索最小相似度
  "maxMemories": 5,              // 每次注入的最大记忆数
  "compactionThreshold": 0.80     // 触发压缩的上下文使用率
}
```

从 [console.supermemory.ai](https://console.supermemory.ai) 获取 API Key。

---

## 4. MCP 服务

### 已配置的 MCP

| MCP | 类型 | 用途 |
|-----|------|------|
| **Context7** | local | 获取最新官方库/框架文档 |
| **MiniMax** | local | 网络搜索 + 图像分析 |

### Context7

按需自动查询最新 API 文档，无需手动触发。

### MiniMax MCP

| 工具 | 功能 |
|------|------|
| `web_search` | 网络搜索，返回结构化结果 |
| `understand_image` | AI 图像分析 |

---

## 5. 技能（Skills）

### Superpowers 技能（14个）

| 技能 | 说明 |
|------|------|
| brainstorming | 创造性工作前头脑风暴 |
| test-driven-development | TDD 开发方法 |
| systematic-debugging | 系统化调试 |
| verification-before-completion | 完成后验证 |
| writing-plans | 编写计划 |
| executing-plans | 执行计划 |
| requesting-code-review | 请求代码审查 |
| receiving-code-review | 接收审查反馈 |
| subagent-driven-development | 子代理驱动开发 |
| dispatching-parallel-agents | 并行代理分发 |
| using-git-worktrees | Git worktree 隔离 |
| finishing-a-development-branch | 完成开发分支 |
| using-superpowers | 使用 superpowers |
| writing-skills | 编写新技能 |

### 用户技能

| 技能 | 说明 |
|------|------|
| fetch4ai | 网页内容抓取 |
| web-research | 结构化网络调研 |
| arxiv | 学术论文检索 |
| pdf | PDF 处理 |
| docx | Word 文档处理 |
| xlsx | 电子表格处理 |
| pptx | PowerPoint 处理 |
| md-to-docx | Markdown 转 Word |
| markitdown | 多格式转 Markdown |
| marp-slides-creator | Marp 幻灯片 |
| frontend-design | 前端界面设计 |
| cc-insights | Claude Code 使用分析 |
| skill-creator | 自定义技能开发 |
| command-development | 斜杠命令开发 |
| chinese-quote-converter | 英文引号转中文 |

---

## 6. 配置详解

### opencode.json（主配置）

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "~/.config/opencode/instructions/language.md"
  ],
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"],
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

### oh-my-opencode.json（Agent 配置）

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/dev/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "oracle": {
      "model": "github-copilot/gpt-5.2",
      "variant": "high"
    },
    "librarian": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "explore": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "multimodal-looker": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "prometheus": {
      "model": "github-copilot/gpt-5.2",
      "variant": "high"
    },
    "metis": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "momus": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "atlas": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
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
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "unspecified-low": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "unspecified-high": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    },
    "writing": {
      "model": "minimax-cn-coding-plan/MiniMax-M2.5"
    }
  },
  "disabled_hooks": [
    "anthropic-context-window-limit-recovery"
  ]
}

```

---

## 7. 使用场景对照表

| 需求 | 推荐方式 | 涉及的插件 |
|------|----------|------------|
| 简单任务 | 直接输入 | Supermemory 记忆偏好 |
| 复杂任务 | `ulw` | OhMyOpenCode + Superpowers |
| 大项目规划 | `@plan` → `/start-work` | OhMyOpenCode 完整流程 |
| 创造性工作 | 先 `skill load superpowers/brainstorming` | Superpowers |
| 写测试 | 自动触发 TDD | Superpowers |
| 调试问题 | 自动触发 systematic-debugging | Superpowers |
| 记住项目知识 | 自动运行 | Supermemory |
| 记住代码风格 | 自动运行 | Supermemory |

---

## 8. 从零复刻本配置

### 前置条件

| 依赖 | 用途 | 安装 |
|------|------|------|
| 现代终端 | Kitty / Ghostty / WezTerm 等 | — |
| GitHub Copilot 订阅 | 提供 Claude / GPT / Gemini 模型 | [github.com/features/copilot](https://github.com/features/copilot) |
| MiniMax Coding Plan | 提供 M2.5 模型 + MCP 工具 | [platform.minimaxi.com](https://platform.minimaxi.com)（国内） |
| uv (Python 包管理器) | 运行 MiniMax MCP Server | `curl -LsSf https://astral.sh/uv/install.sh | sh` |
| Node.js 或 Bun | 安装插件 | — |

### Step 1：安装 OpenCode

```bash
# 推荐：一键安装脚本
curl -fsSL https://opencode.ai/install | bash

# 或 npm
npm install -g opencode-ai

# 或 Homebrew (macOS/Linux)
brew install anomalyco/tap/opencode

# 验证
opencode --version
```

### Step 2：Clone 配置仓库

```bash
# 备份已有配置（如有）
mv ~/.config/opencode ~/.config/opencode.bak 2>/dev/null

# Clone
git clone https://github.com/Joyu42/opencode.git ~/.config/opencode
```

### Step 3：安装 Oh My OpenCode 插件

```markdown
# 推荐方式：交互式安装器
Install and configure oh-my-opencode by following the instructions here:
https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/refs/heads/dev/docs/guide/installation.md
```

### Step 4：安装 Superpowers 插件

```bash
# 1. Clone Superpowers
git clone https://github.com/obra/superpowers.git ~/.config/opencode/superpowers

# 2. 创建目录
mkdir -p ~/.config/opencode/plugins ~/.config/opencode/skills

# 3. 创建符号链接
rm -f ~/.config/opencode/plugins/superpowers.js
ln -s ~/.config/opencode/superpowers/.opencode/plugins/superpowers.js ~/.config/opencode/plugins/superpowers.js
ln -s ~/.config/opencode/superpowers/skills ~/.config/opencode/skills/superpowers
```

### Step 5：安装 Supermemory 插件

1. 注册并获取 API Key：[console.supermemory.ai](https://console.supermemory.ai)

2. 创建 `~/.config/opencode/supermemory.jsonc`：

```jsonc
{
  "apiKey": "sm_你的密钥",
  "similarityThreshold": 0.6,
  "maxMemories": 5,
  "compactionThreshold": 0.80
}
```

### Step 6：编辑 opencode.json

```bash
cat > ~/.config/opencode/opencode.json << 'EOF'
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "~/.config/opencode/instructions/language.md"
  ],
  "mcp": {
    "context7": {
      "type": "local",
      "command": ["npx", "-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"],
      "enabled": true
    },
    "MiniMax": {
      "type": "local",
      "command": ["uvx", "minimax-coding-plan-mcp", "-y"],
      "environment": {
        "MINIMAX_API_KEY": "你的MiniMax Key",
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
EOF
```

### Step 7：认证模型提供商

```bash
# 启动 OpenCode
opencode

# 在 TUI 中使用 /connect 连接 GitHub Copilot
# 按提示完成 OAuth 认证
```

### 安装流程图

```
1. 安装 OpenCode CLI
   ↓
2. Clone 配置仓库 → ~/.config/opencode/
   ↓
3. oh-my-opencode install
   （安装 OhMyOpenCode 插件 + oh-my-opencode.json）
   ↓
4. 安装 Superpowers 插件
   （Clone + 符号链接）
   ↓
5. opencode-supermemory install
   （安装 Supermemory 插件 + 创建 supermemory.jsonc）
   ↓
6. 编辑 opencode.json
   （填入 Context7 / MiniMax API Key）
   ↓
7. opencode → /connect 认证
   ↓
8. 开始使用 ✅
```

---

## 9. 相关链接

- [OpenCode 官方文档](https://opencode.ai/docs/config/)
- [OhMyOpenCode GitHub](https://github.com/code-yeongyu/oh-my-opencode)
- [Superpowers GitHub](https://github.com/obra/superpowers)
- [Supermemory Console](https://console.supermemory.ai)
- [MiniMax 平台](https://platform.minimaxi.com)
- [Context7 MCP](https://mcp.context7.com)

---

*更新日期：2026-03-06*
