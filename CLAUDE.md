# Cloud Manage Agent 项目指引

面向企业的通用 Agent 管理平台(云端托管;沙箱 / Loop / Harness 在云端,本地只做轻量入口)。
本文件随仓库提交,团队成员的 Claude 都会读到。

## Notion 是架构与项目信息的唯一参考源(按需读取)

**不要每次写代码前都读 Notion。** 只在用户明确要求时才读(例如"去读 Notion / 看一下架构 / 按文档来")。

- 任务依赖架构或决策(例如设计模块、改接口、动版本规则)而你不确定时,不要自行去读,先向用户建议读哪一页,用户同意后再读。
- 读的时候只读相关的那一页,不必通读。
- **主动提示(只提示,不自行读)**:每次开始新任务时,以及碰到设计 / 架构类决定前,提示一句"Notion 可能有新内容,要不要读一下?"。其他时候不打扰。

读取方式:用 **Notion MCP 工具**(`notion-fetch`、`notion-search`),不要用浏览器操作 Notion;MCP 做不到的事(分享权限、邀请链接、工作区设置)才用浏览器,并先说明。

### 目录(按需挑一页读)

- 团队空间:`cloud manage agent`(team id `31c2cfb0-68d7-8112-b427-00428143fb05`)
- 总入口:[云管理 Agent 项目文档](https://app.notion.com/p/40a2cfb068d781a19891ddcfab33705a)

| 页面 | 内容 | 链接 |
|---|---|---|
| 01 产品定位与一期需求 | 做什么、一期范围 | https://app.notion.com/p/40a2cfb068d7814d8558ed9749c71de0 |
| 02 系统架构设计 | 整体架构,写代码最相关 | https://app.notion.com/p/40a2cfb068d781c79835f6e98b1f402c |
| 03 版本控制与热更新 | 大小版本、冷启动 / 热更新 | https://app.notion.com/p/40a2cfb068d78198b60af1e071efa5bb |
| 04 记忆、知识库与企业数据沉淀 | 记忆、知识库、数据 | https://app.notion.com/p/40a2cfb068d78174936ed7e5dfb34d79 |
| 05 已有 Agent 接入与可观测性 | 接入、任务 / Token / 成本 | https://app.notion.com/p/40a2cfb068d781a1882bf64a18433a42 |
| 06 竞品与开源项目调研 | 调研 | https://app.notion.com/p/40a2cfb068d781fca4f5f5d0990883f6 |
| 07 会议纪要与待办 | 会议与待办 | https://app.notion.com/p/40a2cfb068d781d0a862d924c625f7e3 |
| 08 待讨论问题 | 未决问题 | https://app.notion.com/p/40a2cfb068d781e59363fff299d8d8dc |
| 09 平台规划问题清单(20 项) | 规划问题 | https://app.notion.com/p/40a2cfb068d78173968bf931b4a357e8 |
| 10 待决策问题(数据库) | 定不下来的问题、选项与推荐 | https://app.notion.com/p/a862001ad3c9457c9b3bfd51ff404d65 |
| 11 实验专区(Codex) | Codex 做的实验、结果与样本 | https://app.notion.com/p/40a2cfb068d78166b1fbec9dcf641b82 |
| 12 模型接入(统一配置) | 统一使用的文本模型接口与 Key | https://app.notion.com/p/40a2cfb068d781c6847fc8419ca648d1 |

**11 实验专区归 Codex**:实验由 Codex 执行并记录(数据库「实验记录」,data source `collection://6db1aaf2-5d07-4361-8d95-536e30897c81`)。Claude 默认只读,用户明确要求才改动;Codex 的规则写在仓库根目录 `AGENTS.md`。

链接打不开时,用 `notion-search` 按标题找。

### 写回 Notion

- 架构、决策、接口、版本规则变化时,更新对应子页,并保持总入口目录同步。
- **禁止**在 GitHub 提交各自版本的架构文档;架构只写 Notion。
- 未确认内容在 Notion 标注"待确认"(例如执行底座 Pi / Dify)。
- 代码与 Notion 冲突或文档过时时,先告诉用户,不要自行偏离。

### 待决策问题(异步决策队列)

讨论中定不下来的问题,统一放进 Notion 数据库「10 待决策问题」(data source `collection://4a052f63-8628-4480-a406-395e58fe2c46`),由用户或同事(及其 Agent)后续决定。

1. **用户当场定不了时**:把问题整理成「问题 / 背景 / 选项(含利弊) / 推荐」列给用户,并问:要不要放进 Notion?
2. **用户点头后才写入**:新建一行,填 问题、状态=`待决定`、谁来定、类别、背景、选项、推荐、提出人、关联文档、提出日期。用户没同意就不写。
3. **读到待决策项时**(用户同意读 Notion 后):查询 状态=`待决定` 或 `讨论中` 的条目,逐条列出,并问用户:这条适合你现在定,还是留给同事(看「谁来定」)?
4. **定下来以后**:填写「最终结论」「决定日期」,状态改为 `已决定`;如果结论影响架构或决策,同步更新对应的 Notion 子页。

## 已确定的核心决策(摘要,以 Notion 为准)

- 纯云端托管;云端大脑 + 本地手脚。
- 执行底座:Pi(Dify 待确认)。
- 工具与 Loop 解耦:Loop 只认 Tools Gateway / Skills Gateway。
- 版本规则:上层 Harness 变动 = 冷启动 + 大版本 + 重启沙箱;下层工具 / Skill 变动 = 热更新 + 小版本 + 不重启沙箱。
- 三个端:网页版、桌面版(员工),管理后台(管理层)。
- 数据集中云端:业务资料 OSS,日志在服务器;权限 公司 → 部门 → 岗位 → 个人。

## 模型接入

- 文本模型统一用阿里云百炼 MaaS 的 OpenAI 兼容接口,说明见 Notion「12 模型接入(统一配置)」。
- 默认文本模型 `qwen3.7-plus`(需要可复现时用快照版 `qwen3.7-plus-2026-05-26`)。
- 代码只从环境变量读 `LLM_BASE_URL`、`LLM_API_KEY`、`LLM_MODEL`;本地放 `.env`(已 gitignore),模板是 `.env.example`。Key 不写进代码、日志或提交。

## 代码与 Git

- 仓库:https://github.com/pans1es/cloud-managed-agents(分支 `main`)。
- Windows + PowerShell:不支持 `&&`,用 `;` 连接命令。
- 推送、开 PR 等对外操作按用户要求进行;GitHub MCP 需用户先通过 `/mcp` 授权。

```powershell
git status
git add -A; git commit -m "说明"
git push
```
