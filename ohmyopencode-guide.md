# OhMyOpenCode 使用指南（个人配置版）

## 当前配置概览

| 角色 | Agent | 模型 | 用途 |
|------|-------|------|------|
| 指挥官 | Sisyphus | `github-copilot/claude-opus-4.6` | 主编排器，规划+委派 |
| 战略规划 | Prometheus | `github-copilot/claude-opus-4.6` | 面试式规划 |
| 军师 | Oracle | `github-copilot/gpt-5.2` | 架构咨询/疑难调试 |
| 侦察兵 | Explore | `minimax-cn/MiniMax-M2.5` | 快速搜索代码库 |
| 侦察兵 | Librarian | `minimax-cn/MiniMax-M2.5` | 搜索文档/开源代码 |
| 参谋 | Metis | `minimax-cn/MiniMax-M2.5` | 计划缺口分析 |
| 参谋 | Momus | `minimax-cn/MiniMax-M2.5` | 方案审查验证 |
| 编排 | Atlas | `minimax-cn/MiniMax-M2.5` | Todo 编排执行 |
| 视觉 | Multimodal Looker | `minimax-cn/MiniMax-M2.5` | 截图/图像分析 |

### Category（子任务分发）

| Category | 模型 | 用途 |
|----------|------|------|
| visual-engineering | `github-copilot/gemini-3.1-pro-preview` | 前端/UI/设计 |
| ultrabrain | `github-copilot/gemini-3.1-pro-preview` | 高难度逻辑 |
| artistry | `github-copilot/gemini-3.1-pro-preview` | 创意/非常规方案 |
| quick | `minimax-cn/MiniMax-M2.5` | 简单小改动 |
| unspecified-low | `minimax-cn/MiniMax-M2.5` | 一般任务 |
| unspecified-high | `minimax-cn/MiniMax-M2.5` | 较复杂任务 |
| writing | `minimax-cn/MiniMax-M2.5` | 文档写作 |

### 设计原则

- **Copilot 额度留给核心 agent**：Sisyphus、Oracle、Prometheus
- **MiniMax 承担大量工作**：搜索、检索、辅助分析、轻量执行，快且无限额
- **特殊任务走 Copilot Gemini**：前端/视觉/高难度推理

---

## 执行流程

### 整体架构

```
你的输入
   ↓
[Intent Gate] — 分析你的真实意图（研究？实现？修复？）
   ↓
[Sisyphus] — 主编排器 (github-copilot/claude-opus-4.6)
   ↓
   ├──→ [Explore] ×N 并行 — 快速搜索代码库 (MiniMax) 🔄
   ├──→ [Librarian] ×N 并行 — 搜索文档/开源代码 (MiniMax) 🔄
   ├──→ [Oracle] — 架构咨询/疑难调试 (GPT-5.2) 💰
   ├──→ [Metis] — 计划缺口分析 (MiniMax) 🔄
   ├──→ [Momus] — 方案审查验证 (MiniMax) 🔄
   ├──→ [Atlas] — Todo 编排执行 (MiniMax) 🔄
   ├──→ [Prometheus] — 战略规划 (Opus 4.6) 💰
   └──→ [Category 子任务] — 按类别分发 ↓
         ├── quick / unspecified / writing → MiniMax 🔄
         └── visual / ultrabrain / artistry → Gemini 3.1 Pro 💰
```

🔄 = MiniMax（快、无限额）　💰 = Copilot（有限额，留给关键任务）

### 典型 ultrawork 执行流

```
1. 你输入 "ultrawork 给项目加个用户认证系统"
2. Sisyphus (Opus) 接收 → Intent Gate 判定为「实现」
3. Sisyphus 同时发射 3-5 个后台任务:
   ├── Explore (MiniMax) → 搜索现有路由、中间件结构
   ├── Explore (MiniMax) → 搜索错误处理模式
   ├── Librarian (MiniMax) → 搜索 JWT 最佳实践文档
   └── Librarian (MiniMax) → 搜索框架认证示例
4. 后台结果返回 → Sisyphus 综合分析
5. Sisyphus 创建 Todo 列表 → 按 Category 分发子任务:
   ├── quick (MiniMax) → 创建配置文件
   ├── unspecified-high (MiniMax) → 实现中间件
   ├── unspecified-high (MiniMax) → 实现路由
   └── ...
6. 遇到架构难题 → 咨询 Oracle (GPT-5.2)
7. 完成后 → 运行诊断验证 → 报告结果
```

---

## 日常使用

### 懒人模式（推荐 90% 场景）

```
ultrawork 给这个项目加个暗黑模式
```

或简写：

```
ulw 重构 auth 模块
```

输入后不用管。Sisyphus 会自动探索代码库、搜文档、拆任务、并行执行、验证结果。

### 精确模式（大项目/关键变更）

1. 按 **Tab** 进入 Prometheus 模式
2. Prometheus 会像工程师一样面试你：问需求、确认范围、识别歧义
3. 生成详细计划
4. 输入 `/start-work` → Atlas 接管，按计划分发子任务执行

### 持续模式（不做完不停）

```
/ulw-loop
```

Ralph Loop 会自我循环执行，直到任务 100% 完成。适合「重构整个测试套件」这类大活。

---

## 性能评估与注意事项

### ✅ MiniMax 充分发挥的场景

- **Explore / Librarian**：最佳位置，官方推荐。快、便宜、可大量并行。
- **quick / unspecified-low / writing**：轻量任务，M2.5 绰绰有余。
- **Atlas**：Todo 编排相对简单，M2.5 能胜任。

### ⚠️ 可能需要关注的场景

- **Metis**（计划缺口分析）：需要一定深度思考，MiniMax 可能偶尔漏掉细微问题，但大部分场景够用。
- **Momus**（方案审查）：官方推荐 GPT-5.2。MiniMax 做审查可能不如 GPT 严格，但省 Copilot 额度是合理的取舍。
- **unspecified-high**：复杂任务走 MiniMax 可能偶尔力不从心。如果发现质量不够，可改回 `github-copilot/claude-sonnet-4.5`。
- **multimodal-looker**（视觉分析）：如果截图/图像分析不工作，改回 `github-copilot/gemini-3-flash-preview`。

### 🔴 绝对不能改的

- **Sisyphus 只用 Claude 家族**：没有 GPT prompt，换成 GPT/MiniMax 会严重退化。
- **Hephaestus 只用 GPT-5.3 Codex**：专为 Codex 构建，不可替换。

### 💡 可进一步优化

- Explore 可尝试 `minimax-cn/MiniMax-M2.5-highspeed`，纯速度场景进一步提速。
- 如果日后购入 Claude Pro/Max，Sisyphus 换成原生 `anthropic/claude-opus-4-6` 体验最佳。

---

## Copilot 额度管理

当前配置把 Copilot 留给了 Sisyphus、Oracle、Prometheus 三个核心 agent。日常 `ultrawork` 使用时大部分调用走 MiniMax，Copilot 消耗不大。

**额度消耗较大的场景**：
- 频繁触发 Oracle 咨询（复杂架构问题）
- 大量前端/视觉任务（走 Gemini 3.1 Pro）
- Prometheus 精确模式的长对话面试

**省额度技巧**：
- 简单任务直接 `ulw`，不走 Prometheus
- 避免反复让 Oracle 解答同类问题

---

## 配置文件位置

- OpenCode 主配置：`~/.config/opencode/opencode.json`
- OmO agent/category 配置：`~/.config/opencode/oh-my-opencode.json`
- 认证信息：`~/.local/share/opencode/auth.json`

---

*生成日期：2026-03-03*
