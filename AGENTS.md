# Quant.Infra.Net.Pro — guide for AI agents / AI agent 使用指南

> For AI agents (Claude Code, Cursor, ...) helping a user install Quant.Infra.Net.Pro and connect to it.
> Once installed, the full guide is inside the app: `quantctl guide` or the MCP tool `get_help`.
> 写给帮助用户安装并连接 Quant.Infra.Net.Pro 的 AI agent。装好之后，完整指南在应用内：`quantctl guide` 或 MCP 工具 `get_help`。
> Answer in the user's language. 请用用户使用的语言回答。

[English](#english) · [中文](#中文)

## English

### Rules for the agent

- **Never place, change or cancel orders.** The MCP server is read-only on purpose. Trading and executing a
  rebalance happen only in the app UI, by the user.
- **Never ask the user to paste secrets into the chat**: Schwab App Key / App Secret / password, license key,
  the MCP token, the remote-panel password. The user types them into the app's own pages. If a command prints a
  secret (for example `quantctl mcp connect`), use it directly in a configuration command and do not repeat it.
- **Never delete or move the user data folder** (see `configure`). Back it up before risky changes:
  copy the whole folder to a dated backup folder.
- **Restarts may need the user.** On Linux under systemd, `sudo systemctl restart quant-infra-net-pro` usually
  needs the user's password; ask the user to run it. Never restart during US market hours if automatic
  rebalance is on, unless the user agrees.
- Prefer the two machine-readable entry points: `quantctl status --json` (works without the app running) and the
  MCP tool `diagnose` (needs the app running with MCP on). Then fix the first failing check.
- This product is self-directed trading infrastructure; do not give investment advice.

### Install

Requirements: a Charles Schwab brokerage account; Schwab Individual Developer access with an **approved** app
(App Key + Secret, created at https://developer.schwab.com); Windows, macOS or Linux (x64). The installers are
self-contained.

Download the latest release from https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public/releases
(Windows `Setup.exe`, Linux `.AppImage`, macOS bundle). To find the newest file names:

```bash
curl -fsSL https://api.github.com/repos/memoryfraction/Quant.Infra.Net.Pro-Public/releases/latest | grep -E '"(tag_name|browser_download_url)"'
```

- **Windows**: run `Setup.exe`; it installs per user and starts the app. The installed copy upgrades itself.
- **Linux desktop or server**: `sudo apt install -y libfuse2` (AppImage needs FUSE), then
  `mkdir -p ~/Applications`, download to `~/Applications/Quant.Infra.Net.Pro.AppImage` (keep this exact name, the
  updater replaces it in place), `chmod +x` it and run it once. For a server that runs all the time, see
  `linux-server`.
- **macOS**: open the bundle; the first start asks for the admin password once to trust the local HTTPS
  certificate.

After the first start, open `https://127.0.0.1:8443/` and continue with `first-run`.
Only download from the GitHub releases page above; never from mirrors.

### First run: EULA, license, Schwab

1. Open `https://127.0.0.1:8443/` and accept the EULA.
2. License: `/start` → enter email → **Verify Email** → open the link in the inbox → back on `/start` click
   Verify Email again → start the **3-month free trial**, or enter an existing key + email in `/settings`.
   An existing license is never revealed by email lookup.
3. `/settings`: the user enters Schwab **App Key**, **App Secret** and **Account Number** (and optionally the
   Schwab login for unattended re-authorization). The Schwab app's callback URL in the developer portal must be
   exactly the app's `Urls` value, by default `https://127.0.0.1:8443/`.
4. Authorize Schwab: `/signin` → log in at Schwab → approve → you return to the dashboard. See `schwab-auth`.
5. `/dashboard` shows account, positions, quotes and orders.

### Linux server (systemd)

1. Install the AppImage as in `install`, as the normal user that will run the app (not root).
2. Put `quantctl` on the PATH: `~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli` (writes
   `~/bin/quantctl`; open a new shell if `~/bin` was not on PATH yet).
3. A system service (needs sudo once), `/etc/systemd/system/quant-infra-net-pro.service`:

```ini
[Unit]
Description=Quant.Infra.Net.Pro
After=network-online.target
Wants=network-online.target

[Service]
User=<user>
Environment=QUANT_INFRA_DATA_DIR=/home/<user>/.config/Quant.Infra.Net.Pro
ExecStart=/home/<user>/Applications/Quant.Infra.Net.Pro.AppImage
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

   `sudo systemctl daemon-reload && sudo systemctl enable --now quant-infra-net-pro`.
   Keep `QUANT_INFRA_DATA_DIR` explicit so data never lands in `/root/.config`. `quantctl` reads the same value
   from the service automatically.
4. Logs: `journalctl -u quant-infra-net-pro -n 100` and `$QUANT_INFRA_DATA_DIR/logs/`.
5. First run on a server: use the remote panel (`remote`) or an SSH tunnel to reach `/`, `/start`, `/settings`
   and `/signin` from the user's PC.
6. Agent access from the PC: the agent can run `ssh <server> ~/bin/quantctl status --json` and
   `ssh <server> ~/bin/quantctl guide <topic>` (non-interactive SSH may not have `~/bin` on PATH); for live data use the MCP tunnel in `mcp`.

### MCP server (read-only)

Off by default. When on, it serves `https://127.0.0.1:8443/mcp` (Streamable HTTP) with a bearer token. It only
accepts connections from the same machine (127.0.0.1); it never goes through the remote panel.

Turn it on — any one of:
- Settings page → **AI assistant (MCP)** → switch on → restart.
- `quantctl mcp on`, then restart the app (Linux service: `sudo systemctl restart quant-infra-net-pro`).
- Add `"Mcp": { "Enabled": true }` to `config/appsettings.user.json` and restart.

Connect a client: `quantctl mcp connect` prints a ready `claude mcp add` command that includes the token
(stored in `config/mcp-token.txt`; treat it like a password). For other clients use URL, transport `http` and
header `Authorization: Bearer <token>`. Rotate the token: stop the app, delete the file, start again.

**App on another machine (Linux server)**: run on the server `quantctl mcp connect --ssh <ssh-host>`; it prints:
1. a tunnel to run on the PC: `ssh -N -L 18443:127.0.0.1:8443 <ssh-host>` (keep it open);
2. how to fetch the server's CA as PEM: `ssh <ssh-host> ~/bin/quantctl mcp ca-pem > ~/.quant-<ssh-host>-ca.pem`, and set
   `NODE_EXTRA_CA_CERTS` to that file for the AI client (Node accepts one file; concatenate PEMs if you need several);
3. the `claude mcp add` command pointing to `https://127.0.0.1:18443/mcp`.

Tools: `get_help`, `diagnose`, `get_recent_log_errors`, `get_account`, `get_balances`, `get_positions`,
`get_orders`, `get_order`, `get_quote`, `get_quotes`, `get_price_history`, `get_option_chain`,
`calculate_rebalance` (suggestions only). Everything a tool returns goes to the AI service the client uses.
`/mcp` returns 403 while the license is invalid or the EULA is not accepted — use `quantctl status` then.

### Troubleshooting

| Symptom | What to check |
|---|---|
| Browser cannot open https://127.0.0.1:8443 | Is the app running? Linux: `systemctl status quant-infra-net-pro`. Port in use by another program: change `Urls` and the Schwab callback URL together. |
| Certificate warning | topic `https`. |
| Redirected to `/start` or `/eula` | License or EULA not done — topic `first-run`. |
| Dashboard says not authorized / expiring | topic `schwab-auth`. |
| Schwab callback fails | callback URL in the Schwab portal ≠ `Urls`, or app not Approved yet. |
| Settings ignored | `config/appsettings.user.json` invalid JSON or two top-level objects (`quantctl status` flags it); restart after edits. |
| `/mcp` 401 | wrong or old token: run `quantctl mcp connect` again. |
| `/mcp` 403 | license/EULA not valid, or the request did not come from 127.0.0.1 (use the SSH tunnel). |
| `/mcp` 404 | MCP is off, or the app was not restarted after turning it on. |
| AI client: certificate error | `NODE_EXTRA_CA_CERTS` → PEM from `quantctl mcp ca-pem`. |
| Remote panel problems | `quantctl doctor`, `quantctl log --tail 30`. |

For errors, the MCP tool `get_recent_log_errors` returns recent redacted warning/error lines.

### Buying, renewing and support

Licenses and renewals: contact the seller (alphawealthlab@outlook.com); renewal rules: https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public#free-trial--renewal.
Community: Telegram group linked in the README.
The long-term home of downloads and public guides is the GitHub repository https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public
(releases, `AGENTS.md` = this guide's install part, `MCP.md`). Prefer it over any website mirror.

## 中文

### agent 必须遵守的规则

- **绝不下单、改单、撤单。** MCP 服务有意设计为只读。交易与执行再平衡只能由用户在应用界面里操作。
- **绝不让用户把机密粘贴到对话里**：Schwab App Key / App Secret / 密码、License Key、MCP token、远程面板口令。
  这些由用户在应用自己的页面里输入。如果某条命令会打印机密（例如 `quantctl mcp connect`），直接把它用在配置命令里，不要在回复中复述。
- **绝不删除或移动用户数据目录**（见 `configure`）。有风险的改动之前先备份：把整个目录复制到带日期的备份目录。
- **重启可能需要用户本人。** Linux 的 systemd 服务执行 `sudo systemctl restart quant-infra-net-pro` 通常需要用户密码，请让用户来执行。
  若开启了自动再平衡，美股交易时段内不要重启，除非用户同意。
- 优先使用两个机器可读入口：`quantctl status --json`（应用没运行也能用）和 MCP 工具 `diagnose`（需应用运行且已开启 MCP）。
  然后从第一个失败项开始修。
- 本产品是自主交易基础设施，不要提供投资建议。

### 安装

前置条件：Charles Schwab 券商账户；Schwab Individual Developer 权限，且应用已 **Approved**（在 https://developer.schwab.com
创建，得到 App Key + Secret）；Windows、macOS 或 Linux（x64）。安装包自带运行时。

从 https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public/releases 下载最新版（Windows `Setup.exe`、Linux `.AppImage`、macOS 应用包）。
查看最新文件名：

```bash
curl -fsSL https://api.github.com/repos/memoryfraction/Quant.Infra.Net.Pro-Public/releases/latest | grep -E '"(tag_name|browser_download_url)"'
```

- **Windows**：运行 `Setup.exe`，按用户安装并自动启动。安装版可自行升级。
- **Linux 桌面或服务器**：`sudo apt install -y libfuse2`（AppImage 需要 FUSE），然后 `mkdir -p ~/Applications`，
  下载为 `~/Applications/Quant.Infra.Net.Pro.AppImage`（文件名保持不变，升级器会原地替换它），`chmod +x` 后运行一次。
  需要常驻运行的服务器见 `linux-server`。
- **macOS**：打开应用包；首次启动会请求一次管理员密码，用于信任本地 HTTPS 证书。

首次启动后打开 `https://127.0.0.1:8443/`，继续看 `first-run`。只从上面的 GitHub releases 页面下载，不要用镜像。

### 首次运行：EULA、License、Schwab

1. 打开 `https://127.0.0.1:8443/`，接受 EULA。
2. License：`/start` → 输入邮箱 → **Verify Email** → 打开邮件中的链接 → 回到 `/start` 再点一次 Verify Email →
   开始 **3 个月免费试用**，或在 `/settings` 输入已有的 Key + 邮箱。已有 License 不会通过邮箱查询泄露。
3. `/settings`：用户填写 Schwab **App Key**、**App Secret**、**Account Number**（可选填 Schwab 登录信息以便无人值守重新授权）。
   开发者门户里 Schwab 应用的回调地址必须与应用的 `Urls` 完全一致，默认 `https://127.0.0.1:8443/`。
4. 授权 Schwab：`/signin` → 在 Schwab 登录 → 同意 → 自动回到仪表盘。见 `schwab-auth`。
5. `/dashboard` 显示账户、持仓、报价与订单。

### Linux 服务器（systemd）

1. 按 `install` 安装 AppImage，使用将来运行应用的普通用户（不要用 root）。
2. 把 `quantctl` 放到 PATH：`~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli`（写入 `~/bin/quantctl`；
   如果 `~/bin` 之前不在 PATH 里，开一个新 shell）。
3. 系统服务（需要一次 sudo），`/etc/systemd/system/quant-infra-net-pro.service`：

```ini
[Unit]
Description=Quant.Infra.Net.Pro
After=network-online.target
Wants=network-online.target

[Service]
User=<用户名>
Environment=QUANT_INFRA_DATA_DIR=/home/<用户名>/.config/Quant.Infra.Net.Pro
ExecStart=/home/<用户名>/Applications/Quant.Infra.Net.Pro.AppImage
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

   `sudo systemctl daemon-reload && sudo systemctl enable --now quant-infra-net-pro`。
   `QUANT_INFRA_DATA_DIR` 要显式写明，避免数据落到 `/root/.config`。`quantctl` 会自动从服务读取同一个值。
4. 日志：`journalctl -u quant-infra-net-pro -n 100` 以及 `$QUANT_INFRA_DATA_DIR/logs/`。
5. 服务器首次运行：用远程面板（`remote`）或 SSH 隧道，从用户电脑访问 `/`、`/start`、`/settings`、`/signin`。
6. agent 从电脑访问：可以运行 `ssh <服务器> ~/bin/quantctl status --json`、`ssh <服务器> ~/bin/quantctl guide <主题>`（非交互 SSH 可能没有把 `~/bin` 加入 PATH）；实时数据用 `mcp` 里的 MCP 隧道。

### MCP 服务（只读）

默认关闭。开启后在 `https://127.0.0.1:8443/mcp`（Streamable HTTP）提供服务，使用 Bearer token 认证。只接受本机（127.0.0.1）连接，
绝不通过远程面板开放。

开启方式（任选其一）：
- 设置页 → **AI assistant (MCP)** → 打开开关 → 重启。
- `quantctl mcp on`，然后重启应用（Linux 服务：`sudo systemctl restart quant-infra-net-pro`）。
- 在 `config/appsettings.user.json` 中加入 `"Mcp": { "Enabled": true }` 并重启。

连接客户端：`quantctl mcp connect` 会打印一条含 token 的现成 `claude mcp add` 命令（token 存于 `config/mcp-token.txt`，按密码对待）。
其他客户端填写 URL、传输方式 `http` 和请求头 `Authorization: Bearer <token>` 即可。轮换 token：停止应用、删除该文件、再启动。

**应用在另一台机器上（Linux 服务器）**：在服务器上运行 `quantctl mcp connect --ssh <ssh 主机名>`，它会打印：
1. 在电脑上运行的隧道：`ssh -N -L 18443:127.0.0.1:8443 <ssh 主机名>`（保持打开）；
2. 以 PEM 格式取回服务器 CA：`ssh <ssh 主机名> ~/bin/quantctl mcp ca-pem > ~/.quant-<ssh 主机名>-ca.pem`，并把 AI 客户端的
   `NODE_EXTRA_CA_CERTS` 指向该文件（Node 只接受一个文件；需要多个时把 PEM 拼接在一起）；
3. 指向 `https://127.0.0.1:18443/mcp` 的 `claude mcp add` 命令。

工具：`get_help`、`diagnose`、`get_recent_log_errors`、`get_account`、`get_balances`、`get_positions`、`get_orders`、`get_order`、
`get_quote`、`get_quotes`、`get_price_history`、`get_option_chain`、`calculate_rebalance`（只给建议）。工具返回的所有内容都会发送给
客户端所用的 AI 服务。License 无效或未接受 EULA 时 `/mcp` 返回 403——此时用 `quantctl status`。

### 排错

| 现象 | 检查什么 |
|---|---|
| 浏览器打不开 https://127.0.0.1:8443 | 应用在运行吗？Linux：`systemctl status quant-infra-net-pro`。端口被占用：同时修改 `Urls` 和 Schwab 回调地址。 |
| 证书警告 | 主题 `https`。 |
| 被跳转到 `/start` 或 `/eula` | License 或 EULA 未完成——主题 `first-run`。 |
| 仪表盘提示未授权 / 即将过期 | 主题 `schwab-auth`。 |
| Schwab 回调失败 | Schwab 门户里的回调地址 ≠ `Urls`，或应用尚未 Approved。 |
| 设置没生效 | `config/appsettings.user.json` 不是合法 JSON 或有两个顶层对象（`quantctl status` 会指出）；修改后需重启。 |
| `/mcp` 401 | token 错误或已过时：重新运行 `quantctl mcp connect`。 |
| `/mcp` 403 | License/EULA 无效，或请求不是来自 127.0.0.1（使用 SSH 隧道）。 |
| `/mcp` 404 | MCP 未开启，或开启后没有重启。 |
| AI 客户端证书错误 | `NODE_EXTRA_CA_CERTS` → `quantctl mcp ca-pem` 生成的 PEM。 |
| 远程面板问题 | `quantctl doctor`、`quantctl log --tail 30`。 |

排查错误时，MCP 工具 `get_recent_log_errors` 返回最近的、已脱敏的警告/错误日志行。

### 购买、续费与支持

License 与续费：联系卖家（alphawealthlab@outlook.com）；续费规则：https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public#free-trial--renewal。
社区：README 中的 Telegram 群。
下载与公开指南的长期地址是 GitHub 仓库 https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public
（releases、`AGENTS.md` = 本指南的安装部分、`MCP.md`）。优先使用它，而不是任何网站镜像。
