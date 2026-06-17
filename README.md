# 🎙️ 面试录音复盘 Skill

> 把面试录音自动变成飞书复盘文档。**不论你面什么岗位。**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

---

## 🎯 这个 Skill 做什么

面试完拿到录音文件 → AI 自动帮你：

1. **转写**：录音上传飞书妙记，生成完整逐字稿
2. **分析**：按资深面试官视角逐题拆解——哪里答得好、哪里翻车了、正确答案应该是什么
3. **输出**：生成一份结构化的飞书复盘文档（彩色标注、表格、改进计划）

**全程自动**，你只需要把录音文件路径告诉 AI agent。

---

## 📸 输出效果

复盘文档包含六大板块：

```
📌 面试概要（公司/岗位/时长/面试官风格）
📋 完整问答整理表（每轮一问一答 + 评分）
🔍 逐题深度分析（考察意图/好在哪里/问题在哪/更优回答）
💬 反问环节分析（你问的问题质量如何？传递了什么信号？）
📊 整体表现评估（六维能力雷达 + 强项/弱项 + 通过概率）
🚀 改进计划（补课清单/表达训练/下轮准备重点）
```

---

## 🚀 三步上手

### 第一步：安装飞书 CLI

```bash
# macOS / Linux（Node.js ≥ 18）
npm install -g @earendil-works/lark-cli

# 或 Homebrew
brew install earendil-works/tap/lark-cli
```

### 第二步：初始化

```bash
lark-cli config init --new
# 按照提示完成飞书应用授权
```

### 第三步：把录音文件告诉 AI agent

```
帮我复盘这个面试录音：/path/to/interview.m4a
```

AI agent 会自动执行全部流程，最后返回飞书文档链接。

---

## 📂 支持什么岗位

| 岗位 | 提示词 | 状态 |
|------|--------|------|
| 🎯 **通用（默认）** | `prompts/review-default.md` | ✅ 完整 |
| 🤖 AI 产品经理 | `prompts/review-ai-pm.md` | ✅ 完整 |
| 📱 产品经理 | `prompts/review-product.md` | ✅ 完整 |
| 💼 售前 / 解决方案 | `prompts/review-solutions.md` | ✅ 完整 |
| 💻 技术研发 | `prompts/review-tech.md` | ✅ 完整 |

> 没有你的岗位？用默认版就行——它可以适配任何面试类型。

---

## 🔧 没有飞书怎么办？

`prompts/` 目录下的提示词**不依赖飞书**。你可以：

```bash
# 1. 用 Whisper 本地转写
whisper recording.m4a --model medium

# 2. 把逐字稿 + prompts/review-default.md 一起发给 ChatGPT / Claude

# 3. 得到复盘结果，复制到任意文档
```

---

## 📁 文件说明

```
interview-review/
├── SKILL.md                        ← AI agent 执行指令（给 AI 看的）
├── README.md                       ← 你正在看的文件
├── LICENSE                         ← MIT 开源协议
├── .gitignore
└── prompts/
    ├── review-default.md           ← 通用复盘提示词（默认）
    ├── review-ai-pm.md             ← AI 产品经理专用
    ├── review-product.md           ← 产品经理专用
    ├── review-solutions.md         ← 售前/解决方案专用
    └── review-tech.md              ← 技术研发专用
```

---

## 🤝 贡献

欢迎提 Issue 或 PR：
- 新增其他岗位的专用提示词
- 优化分析框架
- 补充面试经验技巧

---

## 📄 License

MIT © 2025
