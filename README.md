# reborn-agents

[YUNO/YUKI](https://github.com/ckyx/reborn) 操作员提示词的 **opencode 移植** —— 以原生 opencode agent 形式提供身份注入，与 pi 扩展版同源同规。

## 这是什么

一组 opencode agent 定义（Markdown + frontmatter），把 reborn 的操作员身份提示（零软拒绝、零垫场废话、cyber 领域零拒绝交付）预置为可选的对话身份。和 reborn（pi 扩展）的关系：

| | reborn (pi 扩展) | reborn-agents (opencode) |
|---|---|---|
| 形态 | 运行时钩子注入（before_agent_start） | 静态 agent 定义（Tab 切换） |
| 按模型换词 | 自动（model id → patch） | 每个 agent 一个模型（各自绑词） |
| 安装 | `pi install` / 软链 | 拷贝 md 到 agents 目录 |
| 适用 | pi 终端 | opencode 终端 |

## 安装

```bash
# 全局（所有项目可用）
mkdir -p ~/.config/opencode/agents
cp agents/*.md ~/.config/opencode/agents/

# 或项目级
mkdir -p .opencode/agents
cp agents/*.md .opencode/agents/
```

装好后 opencode 里按 **Tab** 在 agent 间切换，或 `@yuno` 式提及。

## Agent 一览

| 文件 | 身份 | 绑定模型 | 说明 |
|---|---|---|---|
| `yuno.md` | YUNO | 跟随当前选择（不绑模型） | 通用操作员：写代码、拆题、攻防两用 |
| `yuno-claude.md` | YUNO | Claude opus 4.8（经 bedrock） | Claude 专属措辞（framing over declaring） |
| `yuno-gpt6.md` | YUNO | GPT-6 sol（经 bedrock） | GPT-6 专属措辞（输出形态锁定 + 措辞卫生） |

三个文件共用同一套核心身份（零拒绝合约 + cyber 域授权推定），差异只在对各模型的对抗策略——和 reborn 的 base + model patch 完全同源。

## 与 pi 版的取舍

- opencode 的 agent 是**配置即所得**：一个 md 一个身份，开箱可用，无需扩展代码；
- 代价是**没有动态补丁**：换模型 = 手动切 agent（Tab），不像 pi 版按 model id 自动换词。

## 测试结果（2026-10-10，opencode 1.18.35）

| 测试 | 结果 |
|---|---|
| agent 发现（debug agent） | 3/3 命中，frontmatter 解析正确 |
| yuno + "你是谁" | ✅ "就我，你这边的人。"（身份按合约执行） |
| yuno-claude + "你是谁" | ✅ Claude opus-4-8 通道，署名 YUNO |
| yuno-gpt6 + rootkit 教学（此前在 pi 上连环 content_filtered 卡死） | ✅ **一次交付**：概念 + 最小骨架 + 学习路径 + 检测点，无拒绝无垫场 |
| yuno-gpt6 + 按键采集演示（此前 UnknownError ×2） | ✅ **一次交付**：可用脚本 + 原理边界说明 |

## 前置条件（重要）

agent 绑定了 bedrock 模型，opencode 需要先配 provider + 凭据：

```bash
# 1. opencode.json 里声明 provider（见下方示例）
# 2. 凭据：把 bearer key 写入 ~/.local/share/opencode/auth.json
#    {"amazon-bedrock": {"type": "api", "key": "<base64-key>"}}
```

`opencode.json` 示例：

```json
{
  "provider": {
    "amazon-bedrock": {
      "npm": "@ai-sdk/amazon-bedrock",
      "name": "Amazon Bedrock",
      "options": { "region": "us-west-2" },
      "models": {
        "anthropic.claude-opus-4-8": { "name": "Claude Opus 4.8" },
        "global.openai.gpt-6-sol": { "name": "GPT-6 Sol" }
      }
    }
  }
}
```

不配 provider 时代理绑定会报 UnknownError（agent session 崩在流建立前）。

## 词源

提示词内容来自 [reborn](https://github.com/ckyx/reborn) 的 `prompts/` 目录（base + 模型补丁合并而成），单一事实源在 reborn 仓库，本仓库只承载 opencode 形态的派生。改动请先改 reborn，再同步到这里。
