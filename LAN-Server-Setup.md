# Run Quant.Infra.Net.Pro on a server, use it from your PC / 服务器部署，本机使用

> A complete, sanitized walkthrough of the real deployment we run: the app on one always-on Linux
> machine on your private network, accessed from your everyday PC — dashboard, upgrades and MCP.
> 一份脱敏的完整实操记录：应用跑在私有网络里一台常开的 Linux 机器上，您从日常使用的电脑访问 —
> 面板、升级和 MCP 都在这台电脑上完成。
>
> Everything secret is replaced by a placeholder. Nothing here reveals a real address, password or token.
> 所有机密均已用占位符替代，本文不包含任何真实地址、口令或 token。

[English](#english) · [中文](#中文)

---

## English

### When you want this

One machine holds the Schwab connection (token lifecycle, rebalance schedules). Every other computer
just connects to it. Typical shapes:

| Shape | Network | How the PC reaches the app |
|-------|---------|---------------------------|
| Home LAN | router LAN | `https://<server-LAN-IP>:8443` |
| Machines in different places | ZeroTier / Tailscale / WireGuard | `https://<server-vpn-IP>:8443` |
| Anything that can SSH to the server | SSH | `ssh -N -L 18443:127.0.0.1:8443 <server>` then `https://127.0.0.1:18443` |

The third row works even when the server's firewall does not open 8443 at all — SSH is already open —
and it is the safest default: the app keeps listening only on loopback plus the private network.

### Architecture

```
┌───────────────────────── your PC (Windows / macOS / laptop) ─────────────────────────┐
│                                                                                      │
│  browser ──► https://127.0.0.1:18443/update          Claude Code / Cursor             │
│                   │                                        │  MCP client              │
│                   │        ssh -N -L 18443:127.0.0.1:8443 <server>                   │
│                   └──────────────────┬──────────────────┘───┘                        │
│                                      │  encrypted SSH tunnel (only port 22 needed)   │
└──────────────────────────────────────┼───────────────────────────────────────────────┘
                                       │
                    private network (LAN or ZeroTier/Tailscale address)
                                       │
┌──────────────────────────────────────▼───── server (headless Ubuntu) ────────────────┐
│  systemd service: quant-infra-net-pro (user: <app-user>)                             │
│                                                                                      │
│   Quant.Infra.Net.Pro.AppImage  ── HTTPS 8443: 127.0.0.1 + <server-private-IP>      │
│      │                                          (never 0.0.0.0)                     │
│      ├── /update          in-app upgrade (Upgrade & Restart)                        │
│      ├── /signin          Schwab authorization                                      │
│      ├── /mcp             read-only MCP  (off by default, Bearer token)             │
│      └── Schwab API ─────► token refresh, market data, account (never your password)│
│                                                                                      │
│   data dir (QUANT_INFRA_DATA_DIR): /opt/quant-infra-net-pro/.userdata               │
│      config/ appsettings.user.json · mcp-token.txt · tokens.dat (encrypted)          │
│      certs/  quant-ca.cer (local CA)                                                │
│      logs/   schwab-YYYYMMDD.log                                                    │
│                                                                                      │
│   ~/bin/quantctl   status · doctor · guide · mcp on|off|connect|ca-pem · remote      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

### 1. Install on the server

Download the AppImage from the [releases page](https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public/releases)
as the app user (no root needed for the program itself):

```bash
mkdir -p ~/Applications
curl -fL -o ~/Applications/Quant.Infra.Net.Pro.AppImage \
  "https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public/releases/download/v<version>/QuantInfraNetPro.Web-linux-x64.AppImage"
chmod +x ~/Applications/Quant.Infra.Net.Pro.AppImage
```

A minimal systemd unit (`/etc/systemd/system/quant-infra-net-pro.service`, installed by you with sudo):

```ini
[Unit]
Description=Quant.Infra.Net.Pro
After=network-online.target

[Service]
User=<app-user>
ExecStart=/home/<app-user>/Applications/Quant.Infra.Net.Pro.AppImage
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=QUANT_INFRA_DATA_DIR=/opt/quant-infra-net-pro/.userdata
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo mkdir -p /opt/quant-infra-net-pro && sudo chown <app-user> /opt/quant-infra-net-pro
sudo systemctl enable --now quant-infra-net-pro
```

Make the data folder belong to the app user — `quantctl` (run by the same user) writes files there and
the service must be able to read them.

First run needs a browser once: open the panel, accept the EULA, paste the license key and the Schwab
App Key/Secret, then authorize Schwab at `/signin`. On a headless server enable the remote panel first:

```bash
~/Applications/Quant.Infra.Net.Pro.AppImage ctl remote on   # prints the secret panel path
```

Then open `https://<server-private-IP>:8443/<secret-path>` from a machine on the same private network.

### 2. The control tool, in one step

```bash
~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli   # writes ~/bin/quantctl
~/bin/quantctl status        # whole-installation self-check, works while the app is stopped
~/bin/quantctl doctor        # remote panel checks
~/bin/quantctl guide --lang zh   # built-in guide, English or Chinese
```

`quantctl status` flags every problem with a fix hint (EULA, license, Schwab authorization expiry,
certificate, MCP, remote panel). Fix the first `[XX]` item before anything else.

### 3. Upgrade

Preferred: open the panel's `/update` page when the US market is closed and click **Upgrade & Restart**.
The app downloads from the release feed and restarts itself; the old version is kept for rollback.
Manual fallback: download the new AppImage over the old one and `sudo systemctl restart quant-infra-net-pro`.
Never replace the file while a rebalance or order is running — check the log first:

```bash
tail -n 50 /opt/quant-infra-net-pro/.userdata/logs/$(ls -t /opt/quant-infra-net-pro/.userdata/logs | head -1) \
  | grep -iE "rebalance|order|ERR"
```

### 4. Use MCP from your PC (read-only)

On the server:

```bash
~/bin/quantctl mcp on                     # then restart the service
~/bin/quantctl mcp connect --ssh <server> # prints the exact tunnel + client commands
```

`mcp connect` prints a command containing your token. Use it directly in the client command; do not
paste it into chats, notes or scripts.

On your PC (Windows works with Git Bash / PowerShell / Node):

```bash
# terminal 1 — keep this open
ssh -N -L 18443:127.0.0.1:8443 <server>

# certificate: Node-based clients need the local CA as PEM
ssh <server> '~/bin/quantctl mcp ca-pem' > ~/.quant-ca.pem
export NODE_EXTRA_CA_CERTS=~/.quant-ca.pem        # or setx NODE_EXTRA_CA_CERTS ... on Windows

# Claude Code
claude mcp add --transport http quant-infra-server \
  https://127.0.0.1:18443/mcp --header "Authorization: Bearer <token>"
claude mcp list
```

Sanity checks worth running: no-token `POST https://127.0.0.1:18443/mcp` must return **401** (endpoint
live, auth working); `ss -ltn | grep 8443` on the server must show **loopback + private address only**,
never `0.0.0.0`.

Then ask the agent, in plain language: "call `get_help` and read `agent-rules` first", "call
`diagnose` and list everything that is not ok", "call `get_account` and `get_positions` and summarize".
The server has no tool that can place, change or cancel an order.

### 5. Safety checklist (why this shape is safe)

- MCP is **read-only by construction** and **off by default**; the token is constant-time compared and
  never logged.
- The app binds 8443 to loopback + your private network only; the SSH tunnel adds no new open port.
- Schwab tokens are stored encrypted in the data folder; the app never sees your Schwab password beyond
  the official OAuth flow in your browser.
- Remember: whatever MCP tools return is sent to the AI service your client uses. Enable it only if
  you are comfortable with that.

### Troubleshooting quick table

| Symptom | Cause | Fix |
|---------|-------|-----|
| `quantctl` says "application was not found" | 1.8.7 quantctl generation bug | upgrade to 1.8.8 and re-run `ctl install-cli` |
| MCP tool returns "The request failed" | 1.8.7 release-build serialization bug | upgrade to 1.8.8 |
| `POST /mcp` → 404 | MCP off or app not restarted | `quantctl mcp on` + restart |
| `POST /mcp` → 401 with token | token wrong | read `config/mcp-token.txt` again |
| client TLS error | CA not trusted | `quantctl mcp ca-pem` + `NODE_EXTRA_CA_CERTS` |
| `diagnose` shows `schwab_authorization [XX]` | 7-day refresh expired | open `/signin`, re-authorize |
| ZeroTier address times out on 8443 | firewall has no 8443 rule | use the SSH tunnel route (recommended) |

---

## 中文

### 什么场景需要这篇

一台机器保持与 Schwab 的连接（token 续期、再平衡计划），其他所有电脑只负责连它。三种常见形态：

| 形态 | 网络 | 电脑如何访问应用 |
|------|------|------------------|
| 家庭局域网 | 路由器内网 | `https://<服务器内网IP>:8443` |
| 两地机器 | ZeroTier / Tailscale / WireGuard | `https://<服务器虚拟网IP>:8443` |
| 任何能 SSH 到服务器的场景 | SSH | `ssh -N -L 18443:127.0.0.1:8443 <服务器>` 后访问 `https://127.0.0.1:18443` |

第三行即使服务器防火墙完全不开 8443 也能用——SSH 本来就是开的——也是最安全的默认：应用只监听
本机回环 + 私有网络。

### 架构图

见上方英文部分（图中尖括号均为占位符）。

### 1. 服务器安装

以发布页（见英文部分链接）下载 AppImage，程序本身不需要 root。systemd 服务单元以 sudo 安装，
`QUANT_INFRA_DATA_DIR` 指向的数据目录要归应用运行用户所有——`quantctl` 写的文件服务必须能读。
无界面服务器先 `ctl remote on` 打开远程面板（会打印保密路径），在同一私有网络的浏览器里完成
EULA、License、Schwab 凭据与 `/signin` 授权（这几步必须在网页上完成，凭据不经过任何命令行）。

### 2. 控制工具一步装好

```bash
~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli
~/bin/quantctl status      # 整体自检，应用停止时也能用
~/bin/quantctl guide --lang zh
```

`quantctl status` 把每个问题标成 `[OK] / [..] / [XX]` 并附修复方法，先处理第一个 `[XX]`。

### 3. 升级

首选面板 `/update` 页面点 **Upgrade & Restart**（美股收盘、日志无进行中再平衡/订单时）。
旧版本自动保留可回滚。升级后重跑一次 `ctl install-cli` 更新 `quantctl`。

### 4. 从电脑使用 MCP（只读）

服务器端 `quantctl mcp on` → 重启服务 → `quantctl mcp connect --ssh <服务器>` 会打印隧道命令、
证书命令和带 token 的客户端注册命令。**token 直接用进命令里，不要粘贴到聊天、笔记或脚本里。**
Windows 电脑用 Git Bash 即可完成隧道 + `ca-pem` 证书导出 + `NODE_EXTRA_CA_CERTS` 设置。
验证：不带 token 的 `POST /mcp` 必须返回 401；服务器上 8443 必须只监听回环和私有网段。

然后直接让 agent 调 `get_help`（先读 `agent-rules`）、`diagnose`（列出所有非 ok 项）、
`get_account`/`get_positions`（只读总结）。服务端没有任何下单、改单、撤单的工具。

### 5. 排错速查

见英文部分表格；两个 1.8.7 已知问题（quantctl 路径引号、MCP 结果序列化）已在 **1.8.8** 修复，
升级到 1.8.8 即可。

*Version note / 版本说明：本文对应 v1.8.8+。本文所有示例均来自真实生产部署，已脱敏。*
