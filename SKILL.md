---
name: interview-review
version: 2.0.0
description: |
  面试录音通用复盘 Skill。当用户提供面试录音文件（m4a/mp3/mp4 等），
  要求生成面试复盘分析报告并写入飞书文档时触发。
  无需指定岗位类型——默认使用通用复盘模板，也支持 AI产品经理、
  技术研发、售前解决方案、产品经理等专项模板。
  触发关键词：面试录音、面试复盘、面试纪要、面试分析、面试总结。
metadata:
  requires:
    bins: ["lark-cli"]
---

# 面试录音 → 飞书复盘文档（端到端 · 通用版）

> 不论你面的是 AI 产品经理、售前解决方案、技术研发还是任何岗位——
> 本 Skill 将面试录音自动转化为结构化的飞书复盘文档：
> **录音上传 → 妙记转写 → 深度复盘分析 → 飞书文档输出**。

---

## 〇、前置条件：安装飞书 CLI

**如果你还没有安装 `lark-cli`，按以下步骤操作：**

### 0.1 安装 lark-cli

```bash
# 方式一：npm 全局安装（推荐，Node.js ≥ 18）
npm install -g @earendil-works/lark-cli

# 方式二：macOS Homebrew
brew install earendil-works/tap/lark-cli

# 验证安装
lark-cli --version
```

### 0.2 初始化飞书应用配置

你需要一个飞书企业账号，并在飞书开发者后台创建一个自建应用。

```bash
# 发起配置流程（该命令会输出授权链接和二维码）
lark-cli config init --new
```

运行后会输出一个授权链接。**打开链接完成授权**，CLI 会自动保存配置。

> ⚠️ 如果你没有飞书账号或无法创建应用，请看文末的「附录 B：无飞书环境替代方案」。

### 0.3 前置权限一览

本 Skill 需要在飞书开发者后台开通以下权限（scope），并通过 `auth login` 完成用户授权：

| 阶段 | 所需 scope | 用途 |
|------|-----------|------|
| 上传文件 | `drive:drive` | 上传录音到云空间 |
| 生成妙记 | `minutes:minutes` | 将音频转为妙记 |
| 读取逐字稿 | `minutes:minutes:readonly`、`minutes:minutes.artifacts:read`、`minutes:minutes.transcript:export` | 获取转写文本和 AI 摘要 |
| 创建文档 | `docx:document` | 创建飞书文档 |
| 编辑文档 | `docx:document:update` | 追加分析内容 |

> 💡 大部分 scope 在 `config init` 时已开通。如果执行中提示 scope 缺失，AI 会自动引导你授权。

---

## 一、流程全景

```
                        ┌──────────────────────┐
  面试录音 .m4a/.mp3    │  阶段一：录音 → 妙记    │
  ──────────────────▶  │  drive +upload        │
                        │  minutes +upload       │
                        │  vc +notes (逐字稿)    │
                        └──────────┬───────────┘
                                   │ 逐字稿 + AI摘要
                                   ▼
                        ┌──────────────────────┐
                        │  阶段二：逐字稿 → 复盘  │
                        │  Claude 深度分析       │
                        │  逐题评分 / 双向问答    │
                        │  改进计划 / 下级建议    │
                        └──────────┬───────────┘
                                   │ 结构化分析内容
                                   ▼
                        ┌──────────────────────┐
                        │  阶段三：复盘 → 文档   │
                        │  docs +create (骨架)  │
                        │  docs +update (逐段)  │
                        │  飞书 Docx 输出       │
                        └──────────┬───────────┘
                                   │
                                   ▼
                        ┌──────────────────────┐
                        │  阶段四：打开文档      │
                        │  open URL (macOS)    │
                        │  或直接返回链接        │
                        └──────────────────────┘
```

**各阶段产出物：**

| 阶段 | 输入 | 产出 | 文件位置 |
|------|------|------|----------|
| 阶段一 | 面试录音文件 | 逐字稿 TXT + AI 摘要 JSON | `./minutes/{minute_token}/transcript.txt` |
| 阶段二 | 逐字稿全文 | 结构化复盘分析 | 内存 |
| 阶段三 | 复盘分析 | 飞书文档 | 飞书云文档（返回 URL） |
| 阶段四 | 文档 URL | 浏览器打开 | — |

---

## 二、阶段一：录音 → 妙记转写

### 2.1 上传录音到飞书云空间

```bash
# ⚠️ 必须使用相对路径！先 cd 到文件所在目录
cd /path/to/audio/directory
lark-cli drive +upload --file "recording.m4a" --name "面试录音_YYYYMMDD"
```

从返回结果中提取 `data.file_token`，记为 **`<FILE_TOKEN>`**。

### 2.2 生成妙记

```bash
lark-cli minutes +upload --file-token <FILE_TOKEN>
```

从返回结果中提取 `data.minute_url`，取 URL 最后一段作为 **`<MINUTE_TOKEN>`**。

> 示例：`https://xxx.feishu.cn/minutes/obcnXXXX` → `obcnXXXX`

### 2.3 获取逐字稿（需要等待妙记异步生成）

```bash
lark-cli vc +notes --minute-tokens <MINUTE_TOKEN>
```

> ⚠️ 妙记是异步生成的。如果返回 `"minute not ready, try later"`：
> 每 30 秒重试一次，通常 1~3 分钟内就绪。AI 会自动轮询。

成功后返回：
- `artifacts.transcript_file` — 本地逐字稿路径，**务必读取全文**
- `artifacts.summary` — 飞书 AI 摘要（可作为背景参考）
- `artifacts.keywords` — 关键词列表
- `title` — 妙记标题

### 2.4 常见错误处理

| 错误 | 原因 | 解决 |
|------|------|------|
| `unsafe file path` | 用了绝对路径 | `cd` 到文件目录，用相对路径 |
| `minute not ready` | 妙记异步生成中 | 等待 30~60 秒后重试 |
| `missing scope(s)` | 权限不足 | AI 会用 split-flow 引导授权 |
| `exit code 10` | 高风险操作需确认 | AI 会请你确认后加 `--yes` |

---

## 三、阶段二：逐字稿 → 深度复盘分析

### 3.1 选择分析提示词

**首先要确定面试岗位类型**，选择对应的提示词模板：

| 岗位类型 | 提示词文件 | 状态 |
|---------|-----------|------|
| **通用 / 不确定** | `prompts/review-default.md` | ✅ 推荐默认使用 |
| AI 产品经理 | `prompts/review-ai-pm.md` | ✅ 完整 |
| 产品经理 | `prompts/review-product.md` | ✅ 完整 |
| 售前 / 解决方案 | `prompts/review-solutions.md` | ✅ 完整 |
| 技术研发 | `prompts/review-tech.md` | ✅ 完整 |

> 💡 如果用户没有明确说明岗位，默认使用 `prompts/review-default.md`（通用版）。
> 每个提示词都包含该岗位特定的评分维度和考察重点。

**使用前必须读取**对应的提示词文件，理解其分析框架。

### 3.2 提示词共有的分析结构

所有提示词都要求以下六个章节：

1. **面试全景** — 时长、轮次、角色、话题板块、面试风格
2. **完整问答整理表** — 每轮问答的时间线，双向评分
3. **逐题深度分析** — 每道关键题的考察意图、回答质量、评分、更优回答
4. **反问环节分析** — 候选人提问的质量和信号解读
5. **整体表现评估** — 综合得分 + 多维能力分 + 强项/弱项
6. **改进计划** — 知识补课 + 表达训练 + 下轮准备

### 3.3 分析执行原则

AI 必须：

1. **读取完整逐字稿**（不截断，不跳过）
2. **追踪两方角色**：准确区分面试官和候选人的每一句发言
3. **双向评分**：面试官问 → 评候选人回答（1-5 分）；候选人问 → 评提问质量（1-5 分）
4. **引用原文**：每个判断都必须引用逐字稿中的原话，格式为「原文："..."」
5. **给出可复用示例**：对回答不好的问题，提供完整的更优回答范例
6. **识别特殊信号**：面试官的态度变化、重复追问、中途离开、主动给提示等

---

## 四、阶段三：复盘分析 → 飞书文档

### 4.1 创建文档骨架

**核心原则：先建骨架，再分批追加**。一次性写入超长内容容易触发参数限制。

```bash
# 骨架：标题 + callout 概要 + h1 章节标题 + 占位 p
lark-cli docs +create --api-version v2 \
  --content '<title>面试复盘报告 — {公司} · {岗位}</title>
<callout emoji="📌" background-color="light-blue" border-color="blue">
<p><b>概要</b>：{日期/时长/角色/岗位}</p>
</callout>
<h1>一、面试全景</h1><p>（待追加）</p>
<h1>二、完整问答整理表</h1><p>（待追加）</p>
<h1>三、逐题分析</h1><p>（待追加）</p>
<h1>四、反问环节分析</h1><p>（待追加）</p>
<h1>五、整体表现评估</h1><p>（待追加）</p>
<h1>六、改进计划</h1><p>（待追加）</p>'
```

记录返回的 **`<DOC_TOKEN>`** 和 **`<DOC_URL>`**。

### 4.2 分批追加内容

```bash
# 每追加一块用一次命令
lark-cli docs +update --api-version v2 \
  --doc <DOC_TOKEN> \
  --command append \
  --content '<h2>1.1 ...</h2>...'
```

**追加顺序：**
1. 面试全景（表格 + callout）
2. 完整问答表（大表格，含评分）
3. 逐题分析（每道关键题一个 h2 + callout，含亮点/问题/建议）
4. 反问分析
5. 整体评估（得分表 + 列表）
6. 改进计划（补课表 + 训练建议 + 下级准备）

### 4.3 文档样式规范

> ⚠️ 写文档内容前，AI 必须先读取 `lark-doc-xml.md` 和 `lark-doc-style.md`。

**颜色语义（全篇一致）：**

| 语义 | callout 背景色 | 文字色 | emoji |
|------|---------------|--------|-------|
| 亮点/正确 | `light-green` | `green` | ✅ |
| 问题/失分 | `light-red` | `red` | ❌ |
| 建议/改进 | `light-blue` | `blue` | 💡 |
| 注意/警告 | `light-yellow` | `yellow` | ⚠️ |
| 中性/附录 | `light-gray` | — | 📎 |

**必须包含的元素：**
- 开头 `<callout>` 概括核心结论
- 问答表用 `<table>` + 表头 `<th background-color="light-gray">`
- 关键题的更优回答用 `<callout background-color="light-blue">` 高亮
- 章节间用 `<hr/>` 分隔
- 评分数字用 `<span text-color="green/red/yellow">` 着色

### 4.4 末尾附上原始妙记链接

```xml
<callout emoji="📎" background-color="light-gray" border-color="gray">
  <p><b>附：原始妙记链接</b> — <a href="MINUTE_URL">MINUTE_URL</a></p>
</callout>
```

---

## 五、阶段四：打开文档

```bash
# macOS
open "<DOC_URL>"

# 其他系统直接返回链接让用户点击
```

---

## 六、完整执行清单（AI 自检）

AI 在收到面试录音文件后，按此清单逐项执行：

```
☐ 0. 确认 lark-cli 已安装且已配置
☐ 1. cd 到录音目录，用相对路径上传到 Drive
☐ 2. 获取 file_token，调用 minutes +upload 生成妙记
☐ 3. 获取 minute_token，轮询 vc +notes 直到拿到逐字稿
☐ 4. 读取完整逐字稿全文（不截断）
☐ 5. 确认面试岗位类型，读取对应的分析提示词
☐ 6. 按提示词结构生成完整复盘分析（内存中）
☐ 7. docs +create 创建文档骨架
☐ 8. 逐章追加内容（分批 append，避免参数超长）
☐ 9. 末尾附原始妙记链接
☐ 10. open 文档 URL（macOS）或展示链接给用户
☐ 11. 向用户汇报：文件路径、文档链接、核心结论摘要
```

---

## 附录 A：飞书权限授权（split-flow 流程）

当执行中遇到 `missing scope` 错误时，AI 的自动化处理流程：

```bash
# Step 1: 发起授权（不阻塞，立即返回 device_code 和二维码）
lark-cli auth login --scope "<缺失的scope>" --no-wait --json

# Step 2: 从返回中取 verification_url，生成二维码展示给用户
lark-cli auth qrcode "<verification_url>" --ascii

# Step 3: 展示链接 + 二维码，等待用户确认"授权完成"

# Step 4: 用户确认后，用 device_code 完成登录
lark-cli auth login --device-code <device_code>

# Step 5: 重试原命令
```

---

## 附录 B：无飞书环境的替代方案

如果你没有飞书账号或无法安装 lark-cli，可以用纯本地+大模型的方式完成核心流程：

| 阶段 | 飞书方案 | 替代方案 |
|------|---------|---------|
| 语音转文字 | 飞书妙记 | OpenAI Whisper（本地 `whisper` CLI 或 API） |
| AI 摘要 | 妙记内置 AI | 可跳过（直接用逐字稿） |
| 深度复盘 | Claude / 本 Skill 的提示词 | 同样可用，传逐字稿给任意大模型 |
| 输出文档 | 飞书 Docx | 本地 Markdown / HTML / 复制到 Notion |

**本地流程示例：**

```bash
# 1. 语音转文字（需安装 openai-whisper）
whisper recording.m4a --model medium --output_dir ./transcripts

# 2. 将 transcripts/recording.txt + prompts/review-default.md
#    一起发给 ChatGPT / Claude / 其他大模型

# 3. 将输出结果保存为 Markdown，或复制到在线文档
```

> 💡 使用纯本地方案时，`prompts/` 目录下的所有提示词都完全可用——它们不依赖飞书，只依赖一份逐字稿文本。

---

## 附录 C：提示词选择指南

| 如果你面的岗位是... | 用什么提示词 | 特殊评分维度 |
|-------------------|-------------|------------|
| 不确定 / 跨岗位 | `review-default.md` | 沟通表达、逻辑思维、岗位匹配 |
| AI 产品经理 | `review-ai-pm.md` | AI 技术理解、业务翻译、场景嗅觉 |
| 产品经理 | `review-product.md` | 用户洞察、数据分析、优先级决策 |
| 售前 / 解决方案 | `review-solutions.md` | 客户需求挖掘、方案设计、讲标能力 |
| 技术研发（前后端/算法等） | `review-tech.md` | 基础扎实度、系统设计、代码能力 |
