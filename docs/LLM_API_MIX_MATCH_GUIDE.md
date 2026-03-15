# LLM API 混搭配置指南

本指南说明如何自由搭配 Claude Code 执行器和外部审查器的 API。

## 双层架构

```
┌─────────────────────────────────────────────────────┐
│                   Claude Code                        │
│                                                      │
│  ┌─────────────────┐      ┌─────────────────────┐   │
│  │    执行器        │─────▶│      审查器          │   │
│  │  (Executor)     │      │    (Reviewer)       │   │
│  │                 │      │                     │   │
│  │  ANTHROPIC_*    │      │  LLM_*              │   │
│  │  环境变量        │      │  环境变量            │   │
│  └─────────────────┘      └─────────────────────┘   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 执行器配置

执行器通过 `ANTHROPIC_*` 环境变量配置。

### 1. 原生 Claude API
```json
{
  "ANTHROPIC_AUTH_TOKEN": "sk-ant-xxx",
  "ANTHROPIC_BASE_URL": "https://api.anthropic.com",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "claude-opus-4-6"
}
```

### 2. Z.ai (GLM)
```json
{
  "ANTHROPIC_AUTH_TOKEN": "your-zai-key",
  "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
  "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5"
}
```

### 3. Kimi (Moonshot)
官方文档: https://platform.moonshot.cn/docs/guide/agent-support
```json
{
  "ANTHROPIC_AUTH_TOKEN": "sk-xxx",
  "ANTHROPIC_BASE_URL": "https://api.moonshot.cn/anthropic",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "kimi-k2",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "kimi-k2",
  "ANTHROPIC_SMALL_FAST_MODEL": "kimi-k2-thinking-turbo",
  "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "6000"
}
```

### 4. LongCat (美团)
官方文档: https://longcat.chat/platform/docs/zh/ClaudeCode.html
```json
{
  "ANTHROPIC_AUTH_TOKEN": "ak_xxx",
  "ANTHROPIC_BASE_URL": "https://api.longcat.chat/anthropic",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "LongCat-Flash-Thinking-2601",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "LongCat-Flash-Thinking-2601",
  "ANTHROPIC_SMALL_FAST_MODEL": "LongCat-Flash-Lite",
  "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "6000"
}
```

### 5. 自定义兼容端点
```json
{
  "ANTHROPIC_AUTH_TOKEN": "your-key",
  "ANTHROPIC_BASE_URL": "https://your-endpoint.com/anthropic",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "your-model"
}
```

---

## 审查器配置

审查器通过 `llm-chat` MCP 服务器调用任意 OpenAI 兼容 API。

### MCP 服务器配置

在 `~/.claude/settings.json` 中添加：

```json
{
  "mcpServers": {
    "llm-chat": {
      "command": "/usr/bin/python3",
      "args": ["/Users/yourname/.claude/mcp-servers/llm-chat/server.py"],
      "env": {
        "LLM_API_KEY": "your-api-key",
        "LLM_BASE_URL": "https://api.example.com/v1",
        "LLM_MODEL": "model-name"
      }
    }
  }
}
```

### 常用审查器提供商

| 提供商 | LLM_BASE_URL | LLM_MODEL |
|--------|--------------|-----------|
| OpenAI | `https://api.openai.com/v1` | `gpt-4o` |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` |
| MiniMax | `https://api.minimax.chat/v1` | `MiniMax-M2.5` |

---

## 完整配置示例

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your-executor-key",
    "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5"
  },
  "mcpServers": {
    "llm-chat": {
      "command": "/usr/bin/python3",
      "args": ["/Users/yourname/.claude/mcp-servers/llm-chat/server.py"],
      "env": {
        "LLM_API_KEY": "your-reviewer-key",
        "LLM_BASE_URL": "https://api.deepseek.com/v1",
        "LLM_MODEL": "deepseek-chat"
      }
    }
  }
}
```

---

## 使用方式

```bash
# 使用通用 LLM 审查 skill
/auto-review-loop-llm
```

---

## 🔀 替代方案：GLM + MiniMax（使用 Codex MCP）

如果你有 Codex CLI，也可以用 **GLM（智谱）** 做执行者 + **MiniMax-2.5** 做审稿人——同样的跨模型架构，不同的提供商。

| 角色 | 默认 | 替代 |
|------|------|------|
| 执行者（Claude Code） | Claude Opus/Sonnet | GLM-5 / GLM-4.7（智谱 API） |
| 审稿人（Codex MCP） | GPT-5.4 | MiniMax-M2.5（MiniMax API） |

### 第 1 步：安装 Claude Code 和 Codex CLI

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
```

### 第 2 步：配置 `~/.claude/settings.json`

终端输入：`nano ~/.claude/settings.json`

```json
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "your_zai_api_key",
        "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
        "API_TIMEOUT_MS": "3000000",
        "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
        "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
        "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5",
        "CODEX_API_KEY": "your_minimax_api_key",
        "CODEX_API_BASE": "https://api.minimax.chat/v1/",
        "CODEX_MODEL": "MiniMax-M2.5"
    },
    "mcpServers": {
        "codex": {
            "command": "/opt/homebrew/bin/codex",
            "args": [
                "mcp-server"
            ]
        }
    }
}
```

保存：`Ctrl+O` → `Enter` → `Ctrl+X`

### 第 3 步：安装 Skills 并运行

```bash
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
cd Auto-claude-code-research-in-sleep
cp -r skills/* ~/.claude/skills/

# 启动 Claude Code（现在由 GLM 驱动）
claude
```

### 第 4 步：让 GLM 读一遍项目 ⚠️ 重要

> **🔴 不要跳过这一步。** GLM 的 prompt 处理方式与 Claude 不同，必须让 GLM 先读一遍项目，确保 skill 文件能正确解析。

启动 `claude` 后，在对话中输入：

```
读一下这个项目，验证所有 skills 是否正常：
/idea-creator, /research-review, /auto-review-loop, /novelty-check,
/idea-discovery, /research-pipeline, /research-lit, /run-experiment,
/analyze-results, /monitor-experiment, /pixel-art

逐个确认：(1) 能正常加载 (2) frontmatter 解析正确
```

这让 GLM（作为 Claude Code 执行者）先熟悉 skill 文件并提前发现兼容性问题——而不是在跑到一半时才报错。

> ⚠️ **注意：** GLM 和 MiniMax 的行为可能与 Claude 和 GPT-5.4 有所不同。你可能需要调整 skill 中的 `REVIEWER_MODEL` 并微调 prompt 模板以获得最佳效果。核心的跨模型架构不变。
