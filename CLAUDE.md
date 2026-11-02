# Cloud Manage Agent 项目指引

面向企业的通用 Agent 管理平台(云端托管;沙箱 / Loop / Harness 在云端,本地只做轻量入口)。
本文件随仓库提交,团队成员的 Claude 都会读到。

## 语言:统一用简体中文

所有给用户看的文字一律用**简体中文**,包括:回复正文、任务清单(update_plan)、结尾的下一步提示框(wrap_up 的 recap / ask / 按钮文字)、提交说明以外的说明文字、写进 Notion 的内容。
不要夹英文句子,也不要用繁体。代码、命令、文件名、专有名词(如 Agent、Harness、PR)可以保留原文。

## Notion 是架构与项目信息的唯一参考源(按需读取)

**不要每次写代码前都读 Notion。** 只在用户明确要求时才读(例如"去读 Notion / 看一下架构 / 按文档来")。

- 任务依赖架构或决策(例如设计模块、改接口、动版本规则)而你不确定时,不要自行去读,先向用户建议读哪一页,用户同意后再读。
- 读的时候只读相关的那一页,不必通读。
- **主动提示(只提示,不自行读)**:每次开始新任务时,以及碰到设计 / 架构类决定前,提示一句"Notion 可能有新内容,要不要读一下?"。其他时候不打扰。

读取方式:用 **Notion MCP 工具**(`notion-fetch`、`notion-search`),不要用浏览器操作 Notion;MCP 做不到的事(分享权限、邀请链接、工作区设置)才用浏览器,并先说明。
没有原生 Notion 连接器的成员(工作区访客),用 `notion-agent`(集成 Token)读写,见下一节。

### 访客成员如何接入 Notion

访客连不了 Notion 官方原生连接器,改用集成 Token + Notion 官方开源 MCP 服务(`@notionhq/notion-mcp-server`)。

- 前提:工作区所有者已在集成的「内容访问权限」里勾选「云管理 Agent 项目文档」;本机 Node.js ≥ 18;Token 由所有者私下发给你。
- **Token 只放在本机用户级配置里**,不进仓库、`.env`、日志或聊天记录。命令由成员自己在终端运行,不要让 Claude 代填 Token。
- Claude Code(`-s user` 存在用户配置,不会被提交)。Windows 在 **Git Bash** 里运行(PowerShell 会吞掉 `--` 导致报 `unknown option '-y'`;`MSYS_NO_PATHCONV=1` 防止 `/c` 被改写成路径);Mac / Linux 去掉 `MSYS_NO_PATHCONV=1` 和 `cmd /c`:

```bash
MSYS_NO_PATHCONV=1 claude mcp add notion-agent -s user -e NOTION_TOKEN=<你的 Token> -- cmd /c npx -y @notionhq/notion-mcp-server
```

- Codex:在 `~/.codex/config.toml` 加(Windows 的 npx 是 `.cmd` 脚本,直接写 `command = "npx"` 会报 WinError 2,要经 `cmd /c`;Mac / Linux 用 `command = "npx"`、`args = ["-y", "@notionhq/notion-mcp-server"]`):

```toml
[mcp_servers.notion_agent]
command = "cmd"
args = ["/c", "npx", "-y", "@notionhq/notion-mcp-server"]
env = { NOTION_TOKEN = "<你的 Token>" }
```

- 重启后 `/mcp` 里 `notion-agent` 显示 connected,再让 Claude"用 notion-agent 搜索「云管理 Agent 项目文档」,读出目录",能列出 01~12 子页即连通。
- 工具名直接对应 Notion API(如 `API-post-search`、`API-retrieve-a-page`),不是 `notion-fetch` / `notion-search`;读长页面要多调几次。
- 改动在 Notion 里显示为「cloud manage Agent」机器人所做;需要署名时(例如决策结论)在正文写明"由 XX 确认"。

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
- 未确认内容在 Notion 标注"待确认",或放进「10 待决策问题」。
- 代码与 Notion 冲突或文档过时时,先告诉用户,不要自行偏离。

### 待决策问题(异步决策队列)

讨论中定不下来的问题,统一放进 Notion 数据库「10 待决策问题」(data source `collection://4a052f63-8628-4480-a406-395e58fe2c46`),由用户或同事(及其 Agent)后续决定。

1. **用户当场定不了时**:把问题整理成「问题 / 背景 / 选项(含利弊) / 推荐」列给用户,并问:要不要放进 Notion?
2. **用户点头后才写入**:新建一行,填 问题、状态=`待决定`、谁来定、类别、背景、选项、推荐、提出人、关联文档、提出日期。用户没同意就不写。
3. **读到待决策项时**(用户同意读 Notion 后):查询 状态=`待决定` 或 `讨论中` 的条目,逐条列出,并问用户:这条适合你现在定,还是留给同事(看「谁来定」)?
4. **定下来以后**:填写「最终结论」「决定日期」,状态改为 `已决定`;如果结论影响架构或决策,同步更新对应的 Notion 子页。

## 已确定的核心决策(摘要,以 Notion 为准)

- 纯云端托管;云端大脑 + 本地手脚。
- 执行底座:自定义运行时接口(创建会话 / 运行 / 事件流 / 取消 / 恢复),Pi 作为第一个实现;不用 Dify(DQ-11)。
- 工具与 Loop 解耦:Loop 只认 Tools Gateway / Skills Gateway。
- 版本规则:上层 Harness 变动 = 冷启动 + 大版本 + 重启沙箱;下层工具 / Skill 变动 = 热更新 + 小版本 + 不重启沙箱。
- 三个端:网页版、桌面版(员工),管理后台(管理层)。桌面版是否进 1.0、网页端与管理后台是否合并,待决定(见「02 系统架构设计」)。
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
