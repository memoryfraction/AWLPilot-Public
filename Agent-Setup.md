# Let your AI agent install and run AWLPilot / 让 AI agent 帮你安装和使用 AWLPilot

[English](#english) · [中文](#中文)

> This page is for you. The instructions your agent follows are in [AGENTS.md](AGENTS.md).
> 本页写给你本人；agent 照着执行的说明在 [AGENTS.md](AGENTS.md)。
> Other ways to start / 其他上手方式：[one computer / 单机](Quick-start%20Guide.md) · [two computers / 两台电脑](Two-Computer-Setup.md)

---

## English

### What the agent does, and what stays with you

An AI coding agent on your computer (Claude Code, Codex CLI, Cursor, Windsurf or similar) can download, install,
start, check and troubleshoot AWLPilot for you. A few steps need you personally, by design:

| Step | Who |
|---|---|
| Register the Schwab developer app ([guide](Charles%20Schwab%20Developer%20Registration%20Guide.md)) | **You** (needs your identity and a +1 phone) |
| Download the right file from the official releases, install, start | Agent |
| Linux: set up the always-on service | Agent (asks you to type your `sudo` password) |
| Check that everything is healthy (`quantctl status`) and fix the first problem | Agent |
| Click the email verification link for the free trial | **You** |
| Type your Schwab App Key, App Secret and account number into the app's `/settings` page | **You** (never into the chat) |
| Log in at Schwab and approve (and any phone code) | **You** |
| Turn on the read-only MCP server and connect itself to it | Agent |
| Place or execute orders | **You, in the app.** The agent has no tool that can trade. |

### Step 1: copy one of these prompts into your agent

**AWLPilot on this computer:**

```text
Install AWLPilot on this computer for me. Read and follow
https://raw.githubusercontent.com/memoryfraction/AWLPilot-Public/main/AGENTS.md exactly.
Download only from https://github.com/memoryfraction/AWLPilot-Public/releases.
Never ask me to paste passwords, API keys, license keys or tokens into this chat:
tell me which page to type them into. When it is running, turn on its read-only MCP
server, connect yourself to it, run its diagnosis, and tell me what is left for me to do.
```

**AWLPilot on a home server (Linux), used from this PC:**

```text
Install AWLPilot on my Linux server <ssh-host> over SSH. Read and follow
https://raw.githubusercontent.com/memoryfraction/AWLPilot-Public/main/AGENTS.md, section "Linux server".
Download only from https://github.com/memoryfraction/AWLPilot-Public/releases.
Set it up as a systemd service, then give me the SSH tunnel command so I can open it
from this PC at https://127.0.0.1:8443. Never ask me to paste passwords, keys or tokens
into this chat. Finally connect yourself to its read-only MCP server through the tunnel
and run its diagnosis.
```

Replace `<ssh-host>` with the name you use in `ssh <name>`. The full two-computer walkthrough, for when you want
to do it yourself, is [Two-Computer-Setup.md](Two-Computer-Setup.md).

### Step 2: do your part when the agent asks

The agent stops and tells you when it needs you: verify your email, type your Schwab keys into `/settings`, log in
at Schwab on `/signin`. Then it continues: it checks the health report and fixes what it can.

### Step 3: use it with your agent every day

Once connected, ask in plain language. The agent reads your account through the read-only MCP server:

- "How far is my portfolio from my target, and what would I need to buy or sell?"
- "Why does AWLPilot say Schwab is not authorized?"
- "Show me the errors in AWLPilot's log from today."
- "Is there a new AWLPilot version? What changed?"

To act on a suggestion, open AWLPilot and click **Execute Orders** yourself (Pro), or place the orders at Schwab.

### Safety, in short

- Everything the MCP tools return is sent to the AI service your agent uses. Turn MCP off in `/settings` if you do
  not want that.
- The MCP server only answers on the same computer (or through your own SSH tunnel), and needs a token that stays
  in your data folder.
- Use the privacy switch before sharing your screen or screenshots with anyone, including an agent.

---

## 中文

### agent 做什么，哪些留给你

你电脑上的 AI 编程 agent（Claude Code、Codex CLI、Cursor、Windsurf 等）可以替你下载、安装、启动、检查和排错。有几步按设计必须你本人做：

| 步骤 | 谁来做 |
|---|---|
| 注册嘉信开发者应用（[指南](Charles%20Schwab%20Developer%20Registration%20Guide.md)） | **你**（需要本人身份和 +1 手机号） |
| 从官方 Releases 下载对应文件、安装、启动 | agent |
| Linux：配置常驻服务 | agent（需要 `sudo` 时请你输入密码） |
| 检查状态（`quantctl status`）并修复第一个问题 | agent |
| 点击邮箱里的验证链接，开始免费试用 | **你** |
| 在软件的 `/settings` 页面填入嘉信 App Key、App Secret 和账号 | **你**（绝不发到聊天里） |
| 在嘉信页面登录、授权（以及手机验证码） | **你** |
| 打开只读 MCP 服务，并把自己连上去 | agent |
| 下单或执行再平衡 | **你本人，在软件里操作。** agent 没有任何能交易的工具。 |

### 第 1 步：把下面一段提示词复制给 agent

**装在这台电脑上：**

```text
帮我在这台电脑上安装 AWLPilot。严格按照
https://raw.githubusercontent.com/memoryfraction/AWLPilot-Public/main/AGENTS.md 执行。
只从 https://github.com/memoryfraction/AWLPilot-Public/releases 下载。
不要让我把密码、API Key、License Key 或令牌贴到聊天里，告诉我该填到哪个页面。
装好运行后，打开它的只读 MCP 服务，把你自己连上去，运行诊断，再告诉我还剩哪些事要我做。请用中文回答。
```

**装在家里的 Linux 服务器上，在这台电脑上用：**

```text
通过 SSH 在我的 Linux 服务器 <ssh-host> 上安装 AWLPilot。按照
https://raw.githubusercontent.com/memoryfraction/AWLPilot-Public/main/AGENTS.md 的 "Linux server" 一节执行。
只从 https://github.com/memoryfraction/AWLPilot-Public/releases 下载。
把它配置成 systemd 服务，然后给我 SSH 隧道命令，让我在这台电脑上用 https://127.0.0.1:8443 打开。
不要让我把密码、Key 或令牌贴到聊天里。最后通过隧道连上它的只读 MCP 服务并运行诊断。请用中文回答。
```

把 `<ssh-host>` 换成你平时 `ssh <名字>` 用的名字。想自己动手的完整步骤见[两台电脑教程](Two-Computer-Setup.md)。

### 第 2 步：agent 叫你时，完成你的部分

需要你时 agent 会停下来告诉你：验证邮箱、在 `/settings` 填嘉信 Key、在 `/signin` 登录嘉信。完成后它会继续检查状态报告，能修的自己修。

### 第 3 步：日常用 agent 问

连上之后直接用大白话问，agent 通过只读 MCP 读取你的账户：

- "我的组合离目标差多少？要买卖什么？"
- "AWLPilot 为什么说嘉信没授权？"
- "看看 AWLPilot 今天日志里的错误。"
- "AWLPilot 有新版本吗？改了什么？"

要照建议执行，请你自己打开 AWLPilot 点 **Execute Orders**（Pro），或到嘉信自己下单。

### 安全要点

- MCP 工具返回的所有内容都会发给你的 agent 所用的 AI 服务。不想这样，就在 `/settings` 关掉 MCP。
- MCP 服务只接受本机（或你自己的 SSH 隧道）的连接，并且需要一个保存在你数据文件夹里的令牌。
- 给任何人（包括 agent）共享屏幕或截图前，先打开隐私开关。
