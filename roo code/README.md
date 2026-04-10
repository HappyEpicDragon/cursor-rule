# Roo Code 配置指南

## 文件结构

将以下文件复制到你的**项目根目录**：

```
your-project/
├── .roomodes                          # 模式定义（5个自定义模式）
└── .roo/
    ├── rules-lead/
    │   └── 01-main-rules.md           # 主 Agent 规则（含硬件信息、委派逻辑、会话管理）
    ├── rules-implementer/
    │   └── 01-rules.md                # 代码实现规则
    ├── rules-verifier/
    │   └── 01-rules.md                # 代码验收规则（只读）
    ├── rules-monitor/
    │   └── 01-rules.md                # 实验监控规则（含 tmux 模板）
    └── rules-plotter/
        └── 01-rules.md                # 科研绘图规则
```

## 配置步骤

### 1. 复制文件

```bash
# 在项目根目录执行
cp -r /path/to/roo-code-config/.roomodes .
cp -r /path/to/roo-code-config/.roo .
```

### 2. 配置 API Provider

打开 VS Code → Roo Code 面板 → 设置（齿轮图标）→ API Provider

推荐配置：
- **主 Agent (lead)**：Claude Sonnet 4.6 或 Gemini 2.5 Pro（需要强推理能力）
- **implementer**：DeepSeek Coder V3 或 Qwen2.5-Coder（便宜且代码能力强）
- **verifier**：DeepSeek V3 或 Gemini 2.5 Flash（只读审查，不需要最强模型）
- **monitor**：DeepSeek V3（脚本模板化，不需要强模型）
- **plotter**：DeepSeek Coder V3（matplotlib 代码生成）

#### 通过 OpenRouter（推荐，一个 Key 访问所有模型）

1. 去 [openrouter.ai](https://openrouter.ai) 注册并充值
2. 获取 API Key
3. Roo Code 设置 → API Provider 选 "OpenRouter"
4. 填入 Key

#### 通过 BYOK 直连各供应商

分别配置 Anthropic / DeepSeek / 其他供应商的 API Key。

### 3. 设置 Per-Mode 模型

切换到每个模式，选择对应模型，Roo Code 会自动记住（Sticky Models 功能）：

1. 点击聊天框下方的模式选择器 → 选 "🧠 Lead Architect" → 选择 Sonnet 4.6
2. 切换到 "⚡ Code Implementer" → 选择 DeepSeek Coder V3
3. 切换到 "🔍 Code Verifier" → 选择 DeepSeek V3
4. 切换到 "🔬 Experiment Monitor" → 选择 DeepSeek V3
5. 切换到 "📊 Research Plotter" → 选择 DeepSeek Coder V3

之后每次切换模式，模型会自动跟随。

### 4. 配置自动审批（可选，提升效率）

Roo Code 默认每次文件操作都需要你手动批准。对于信任的子模式，可以开启自动审批：

设置 → Auto Approve：
- **implementer**：可以开启 write 和 command 的自动审批
- **monitor**：可以开启 command 的自动审批（它需要频繁执行 shell 命令）
- **verifier**：无需配置（只读模式）
- **plotter**：可以开启 write 和 command 的自动审批

## 使用方式

### 日常工作流

1. 打开 Roo Code，默认进入 **Lead Architect** 模式
2. 描述你要做的事情，例如："我要实现一个新的数据增强模块"
3. Lead 会拆解任务，通过 `new_task` 委派给对应子模式
4. 子模式完成后通过 `attempt_completion` 返回结果
5. Lead 审核结果，决定下一步

### 手动切换模式

如果你想直接跟某个子模式对话：
- 点击模式选择器切换
- 或输入 `/implementer`、`/verifier` 等 slash command

### Cursor → Roo Code 概念映射

| Cursor 概念 | Roo Code 对应 |
|-------------|---------------|
| `.mdc` 文件 | `.roomodes` + `.roo/rules-{slug}/` |
| `alwaysApply: true` | rules 目录下的文件自动应用于对应模式 |
| 子 Agent 调度 | `new_task` 工具 + 模式切换 |
| `readonly: true` | `groups: [read]`（只给 read 权限） |
| `is_background: true` | Roo Code 暂无原生后台模式，通过 tmux 实现 |
| `model: fast` | Sticky Models（每个模式绑定不同模型） |
