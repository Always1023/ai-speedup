# ai-speedup

让 AI / 模型 / Agent 任务更快——**先诊断瓶颈，再选对提速手段**，而不是盲目堆并发。

## 这是什么

一个 WorkBuddy Skill。它判断任务属于哪种情况，再选对应的提速方式：

- **多件独立任务** → 并发 / 批处理（提吞吐）
- **单条 + I/O 等待**（等接口/文件/网络）→ 异步 + 流式 + 连接池
- **单条 + 算力瓶颈**（模型逐 token 生成）→ 流式改善体感；能拆则转并发，不能拆就保持流式

附带可直接用的 Prompt 库和 Python / JS 代码片段。

## 核心心法

> 并发提升的是**吞吐量**，不是**单条延迟**。单条回复慢，多半是模型在逐字生成——这时并发无效，用流式改善体感即可。

## 目录结构

```
ai-speedup/
├── SKILL.md                     # 决策框架与选型规则
└── references/
    ├── prompts.md               # 总控指挥官 + 4 个分场景 Prompt
    └── code_examples.md         # 流式 / 异步并发 / 连接池 代码
```

## 安装

把本仓库内容放到 WorkBuddy 的用户级 skills 目录：

- Windows：`C:\Users\<你>\.workbuddy\skills\ai-speedup\`
- macOS / Linux：`~/.workbuddy/skills/ai-speedup/`

即最终路径为 `.../skills/ai-speedup/SKILL.md`，重开会话后自动识别。

## 使用

- 同账号的 WorkBuddy 项目会自动触发；
- 也可直接把 `references/prompts.md` 里的「总控指挥官」当作 System Prompt 粘贴给任意 Agent 使用。
