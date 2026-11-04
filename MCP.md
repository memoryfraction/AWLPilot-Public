# MCP Server Guide / MCP 使用说明

> Applies to / 适用版本：Quant.Infra.Net.Pro v1.8.4+ (help tools, Settings switch and `quantctl mcp`: v1.8.7+ / 帮助工具、设置页开关与 `quantctl mcp`：v1.8.7+)

---

## English

Quant.Infra.Net.Pro can let AI clients (Claude Code, Cursor, ...) **read** your Schwab data through the Model
Context Protocol (MCP). The server runs inside the app, on your own machine, at
`https://127.0.0.1:8443/mcp`. **It is off by default and it is read-only: it cannot place, change or cancel
any order.**

### 1. Turn it on

Easiest (v1.8.7+): **Settings → AI assistant (MCP)** → tick the switch → Apply → restart; or run
`quantctl mcp on` and restart. Or edit your user settings file and restart the app:

| OS | File |
|----|------|
| Windows | `%APPDATA%\Quant.Infra.Net.Pro\config\appsettings.user.json` |
| macOS / Linux | `~/.config/Quant.Infra.Net.Pro/config/appsettings.user.json` |

```json
{
  "Mcp": {
    "Enabled": true
  }
}
```

(If you set `QUANT_INFRA_DATA_DIR`, the file is under that folder instead.) Merge it into the existing
JSON; do not create a second top-level object.

| Setting | Default | Meaning |
|---------|---------|---------|
| `Mcp:Enabled` | `false` | Master switch. When `false`, `/mcp` does not exist (404) and no token is created. |
| `Mcp:MaxResults` | `200` | Upper bound for list-style results (longer lists are truncated and flagged). |

### 2. Get your token

On first start the app creates a random token in `config/mcp-token.txt` (same folder as above). Every request
must send `Authorization: Bearer <token>`. **The token gives read access to your brokerage data. Never share
it.** To rotate it, stop the app, delete the file, start the app again.

### 3. Connect a client

Claude Code:

```bash
claude mcp add --transport http quant-infra https://127.0.0.1:8443/mcp --header "Authorization: Bearer <token>"
```

v1.8.7+: `quantctl mcp connect` (or Settings → AI assistant → Show connection command) prints this command with
your token filled in. The app on a Linux server: run `quantctl mcp connect --ssh <host>` there; it prints the SSH
tunnel to open on your PC and the matching commands.

Any client that accepts a remote HTTP MCP server takes the same three values: URL, transport `http`
(Streamable HTTP) and the `Authorization` header. For other clients follow their own MCP documentation.

The certificate is issued by a local CA. If a client complains about it, trust the CA file
`certs/quant-ca.cer` in the data folder. Node-based clients need `NODE_EXTRA_CA_CERTS` pointing to a **PEM** copy
(the `.cer` file is DER): `quantctl mcp ca-pem > quant-ca.pem` (v1.8.7+).

### 4. Tools (all read-only)

Help (v1.8.7+): `get_help` (install / configure / use guide, English or Chinese), `diagnose` (self-check with fix
hints, never returns secret values), `get_recent_log_errors` (redacted). Ask your agent, for example,
"check my setup" or "帮我检查一下配置".

Data: `get_account`, `get_balances`, `get_positions`, `get_orders`, `get_order`, `get_quote`, `get_quotes`,
`get_price_history`, `get_option_chain`, `calculate_rebalance`.

`calculate_rebalance` only computes suggestions. Executing a rebalance, like any trading, is done in the app.

### 5. Safety model

- Bound to `127.0.0.1`; requests with a non-local `Origin` or `Host` are rejected (403).
- Token comparison is constant-time; the token is never logged or returned by a tool.
- There is no tool that places, changes or cancels orders.
- **Everything the tools return is sent to whichever AI service your client uses.** Only enable MCP if you are
  comfortable with your account, positions, orders and quotes being seen by that service.

---

## 中文

Quant.Infra.Net.Pro 可以通过 Model Context Protocol（MCP）让 AI 客户端（Claude Code、Cursor 等）**读取**
您的 Schwab 数据。服务运行在应用内、您自己的电脑上，地址为 `https://127.0.0.1:8443/mcp`。
**默认关闭，且只读：无法下单、改单或撤单。**

### 1. 开启

最简单（v1.8.7+）：**设置页 → AI assistant (MCP)** → 勾选开关 → Apply → 重启；或运行 `quantctl mcp on` 后重启。
也可以编辑用户配置文件并重启应用：

| 系统 | 文件 |
|------|------|
| Windows | `%APPDATA%\Quant.Infra.Net.Pro\config\appsettings.user.json` |
| macOS / Linux | `~/.config/Quant.Infra.Net.Pro/config/appsettings.user.json` |

```json
{
  "Mcp": {
    "Enabled": true
  }
}
```

（如设置了 `QUANT_INFRA_DATA_DIR`，则文件位于该目录下。）请合并进已有 JSON，不要新建第二个顶层对象。

| 配置项 | 默认值 | 含义 |
|--------|--------|------|
| `Mcp:Enabled` | `false` | 总开关。为 `false` 时 `/mcp` 不存在（404），也不会生成 token。 |
| `Mcp:MaxResults` | `200` | 列表类结果的上限（更长的列表会被截断并标记）。 |

### 2. 获取 token

首次启动时应用会在 `config/mcp-token.txt`（同上目录）生成随机 token。每个请求都必须携带
`Authorization: Bearer <token>`。**token 等同于对您券商数据的读取权限，请勿分享。**
轮换方法：停止应用、删除该文件、再次启动。

### 3. 连接客户端

Claude Code：

```bash
claude mcp add --transport http quant-infra https://127.0.0.1:8443/mcp --header "Authorization: Bearer <token>"
```

v1.8.7+：`quantctl mcp connect`（或设置页 → AI assistant → Show connection command）会打印已填好 token 的这条命令。
应用在 Linux 服务器上时：在服务器上运行 `quantctl mcp connect --ssh <主机>`，它会打印需要在电脑上打开的 SSH 隧道及对应命令。

任何支持远程 HTTP MCP 服务的客户端都只需要这三项：URL、传输方式 `http`（Streamable HTTP）、
`Authorization` 请求头。其他客户端请参阅其自身的 MCP 文档。

证书由本地 CA 签发。如客户端报证书错误，请信任数据目录下的 `certs/quant-ca.cer`
。基于 Node 的客户端需要把 `NODE_EXTRA_CA_CERTS` 指向 **PEM** 格式副本（`.cer` 是 DER 格式）：
`quantctl mcp ca-pem > quant-ca.pem`（v1.8.7+）。

### 4. 工具（全部只读）

帮助（v1.8.7+）：`get_help`（中英双语的安装 / 配置 / 使用指南）、`diagnose`（带修复提示的自检，绝不返回机密值）、
`get_recent_log_errors`（已脱敏）。可以直接对 agent 说“帮我检查一下配置”或 "check my setup"。

数据：`get_account`、`get_balances`、`get_positions`、`get_orders`、`get_order`、`get_quote`、`get_quotes`、
`get_price_history`、`get_option_chain`、`calculate_rebalance`。

`calculate_rebalance` 只计算建议。执行再平衡以及任何交易都在应用内操作。

### 5. 安全模型

- 仅绑定 `127.0.0.1`；`Origin` 或 `Host` 非本机的请求会被拒绝（403）。
- token 采用常量时间比较，绝不写入日志，也不会被任何工具返回。
- 不存在任何下单、改单或撤单的工具。
- **工具返回的所有内容都会发送给您的客户端所使用的 AI 服务。** 仅当您能接受该服务看到您的账户、持仓、订单和报价时才开启 MCP。
