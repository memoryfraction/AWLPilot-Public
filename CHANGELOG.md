# Changelog

All notable changes to Quant.Infra.Net.Pro are documented in this file.
本文件记录 Quant.Infra.Net.Pro 的所有重要变更。

This file is the single source of truth for release notes. It feeds three consumers:
本文件是发布说明的唯一来源，服务三个消费者：

1. `vpk pack --releaseNotes` — the current version section becomes the updater's release notes.
2. `VersionHistory.razor` — embedded as a resource instead of a hand-maintained list.
3. `README.md` version table — kept in sync from here.

Versioning follows `vX.Y.Z` git tags; the tag is the authoritative version.
Section headings use the fixed format `## [X.Y.Z] - YYYY-MM-DD` (or `- Unreleased`).
版本号遵循 `vX.Y.Z` git tag，tag 是权威版本。章节标题统一使用
`## [X.Y.Z] - YYYY-MM-DD`（或 `- Unreleased`）格式。

---

## [1.8.8] - 2026-11-03

### English

**Fix release: every MCP tool now works in the published (obfuscated) build.**

- **Fixed: MCP tools returned "The request failed" in the release build.** `get_help`, `diagnose` and
  `get_recent_log_errors` serialized their results with anonymous types; Obfuscar strips the constructor
  parameter names of anonymous types, so System.Text.Json threw inside the tool. The affected payloads are now
  built with the named-field helper (`McpJson.Fields`), the same pattern the account tools already used.
- **Fixed: `ctl install-cli` wrote a broken `~/bin/quantctl`.** The generated script embedded the default
  program path as `${QUANT_APPIMAGE:-'<path>'}`; the inner single quotes were literal characters, so the
  `[ -x "$APP" ]` check always failed and quantctl reported "the application was not found" even when the
  AppImage existed. The generated script now assigns a plain quoted path and honours `QUANT_APPIMAGE`.
  Existing broken installs are fixed by re-running `Quant.Infra.Net.Pro.AppImage ctl install-cli`.

### 中文

**修复版：发布版（混淆后）中所有 MCP 工具现在都能正常工作。**

- **修复：MCP 工具在发布版中返回 "The request failed"。** `get_help`、`diagnose`、`get_recent_log_errors`
  此前用匿名类型序列化结果；Obfuscar 会抹掉匿名类型构造函数的参数名，导致 System.Text.Json 在工具内部抛异常。
  相关负载改用命名字段辅助方法（`McpJson.Fields`），与账户工具原有写法一致。
- **修复：`ctl install-cli` 生成的 `~/bin/quantctl` 无法运行。** 生成的脚本把默认程序路径写成
  `${QUANT_APPIMAGE:-'<路径>'}`，内层单引号是字面字符，`[ -x "$APP" ]` 永远失败，即使 AppImage 存在
  quantctl 也报"application was not found"。新脚本改为普通引号赋值并支持 `QUANT_APPIMAGE`。
  已受影响的安装重跑一次 `Quant.Infra.Net.Pro.AppImage ctl install-cli` 即可修复。

---

## [1.8.7] - 2026-11-03

### English

**New: let an AI agent help you install, configure and use the app — in English or Chinese.**

- **Built-in guide for AI agents** — install, first run, configuration, Schwab authorization, certificates,
  MCP, remote panel, Linux server, upgrade and troubleshooting, in English and Chinese. Read it with the new MCP
  tool `get_help` or with `quantctl guide`.
- **Self-check** — the new MCP tool `diagnose` and the new command `quantctl status [--json]` report the version,
  data folder, settings file, EULA, license, Schwab credentials, Schwab authorization expiry, certificate, MCP and
  remote panel, each with a fix hint. They only say whether a value is set, never the value.
  `quantctl status` works while the app is stopped and without a valid license.
- **Recent log problems** — the new MCP tool `get_recent_log_errors` returns recent warning and error lines with
  tokens, account identifiers, emails and amounts redacted.
- **Turning MCP on is easier** — a switch in Settings → AI assistant (MCP) with a ready-to-copy Claude Code
  command, or `quantctl mcp on` / `off`. `quantctl mcp connect` prints the connection command;
  `quantctl mcp connect --ssh <host>` prints the SSH tunnel and commands for an app running on a Linux server;
  `quantctl mcp ca-pem` prints the local CA as PEM for `NODE_EXTRA_CA_CERTS`.
- **Linux: `quantctl` in one step** — `Quant.Infra.Net.Pro.AppImage ctl install-cli` writes `~/bin/quantctl`.
- **Still read-only** — no tool places, changes or cancels orders.
- **Links now point to GitHub** — the in-app guide, the Dashboard "Renew Subscription" button and the EULA page link
  to https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public, the long-term home of downloads and guides, instead of a website.

### 中文

**新增：让 AI agent 用中文或英文帮您安装、配置和使用本应用。**

- **内置 AI agent 指南** —— 安装、首次运行、配置、Schwab 授权、证书、MCP、远程面板、Linux 服务器、升级与排错，中英双语。
  可通过新的 MCP 工具 `get_help` 或 `quantctl guide` 阅读。
- **自检** —— 新的 MCP 工具 `diagnose` 与新命令 `quantctl status [--json]` 报告版本、数据目录、设置文件、EULA、License、
  Schwab 凭据、Schwab 授权到期时间、证书、MCP 与远程面板，每项附修复提示。只说明某个值是否已设置，绝不输出值本身。
  `quantctl status` 在应用停止、License 无效时也能用。
- **最近的日志问题** —— 新的 MCP 工具 `get_recent_log_errors` 返回最近的警告与错误日志行，token、账户标识、邮箱和金额均已脱敏。
- **开启 MCP 更简单** —— 设置页 → AI assistant (MCP) 提供开关和可直接复制的 Claude Code 命令，或使用 `quantctl mcp on` / `off`。
  `quantctl mcp connect` 打印连接命令；`quantctl mcp connect --ssh <主机>` 为运行在 Linux 服务器上的应用打印 SSH 隧道与命令；
  `quantctl mcp ca-pem` 以 PEM 格式输出本地 CA，供 `NODE_EXTRA_CA_CERTS` 使用。
- **Linux：一步装好 `quantctl`** —— `Quant.Infra.Net.Pro.AppImage ctl install-cli` 写入 `~/bin/quantctl`。
- **仍然只读** —— 没有任何下单、改单或撤单的工具。
- **链接改为指向 GitHub** —— 内置指南、仪表盘的 “Renew Subscription” 按钮和 EULA 页面改为链接到下载与指南的长期地址
  https://github.com/memoryfraction/Quant.Infra.Net.Pro-Public，不再指向网站。

---

## [1.8.6] - 2026-11-01

### English

**Security hardening.**

### 中文

**强化安全。**

---

## [1.8.5] - 2026-10-22

### English

**New: optional remote panel — use the app from another computer on your own network.**

- **Reach it from any computer on your network** — a home or office LAN, or a virtual network such as
  ZeroTier, Tailscale or WireGuard. Open `https://<server address>:<port>/<secret path>`, type the password, and
  you get the same pages you get on the server itself. No remote desktop needed. **Off by default.**
- **Password sign-in** — a password is required for everything. A random one is generated when you turn it on;
  only a hash is stored. Repeated wrong guesses slow down, changing the password signs everyone out.
- **Hidden behind a secret path** — a wrong path looks like a page that does not exist (404). Only computers
  inside the network you allow can connect at all.
- **HTTPS first** — the certificate covers the server's network addresses; trust it once per device from the
  `/lan/ca` page. Plain HTTP is available if you choose it, with a warning.
- **`quantctl` command** — on the server, `quantctl info`, `remote on|off`, `passwd`, `port`, `scheme`,
  `entrance`, `bind`, `trust-ca`, `doctor` and `log` show and change everything.
  Quick start: run `quantctl remote on`, restart the app, then `quantctl info` prints the address and
  `quantctl doctor` checks the setup.
- **Authorize Schwab from another computer** — a new page (`/lan/finish-auth`) takes the address your browser
  shows after you sign in to Schwab, so the whole authorization can be done remotely.
- **Safer by default** — the app never listens on the network unless you switch the remote panel on.
  If a settings file names a non-loopback address in `Urls`, the app now listens on this computer only and says so
  in the log (that setup had no login). The Schwab callback address is now always the **first** address in `Urls`.

### 中文

**新增：可选的远程面板——在您自己网络里的另一台电脑上使用本应用。**

- **同一网络内任意电脑都能访问** —— 家庭或办公室局域网，或 ZeroTier、Tailscale、WireGuard 等虚拟内网。
  打开 `https://<服务器地址>:<端口>/<入口路径>`，输入口令，即可看到与服务器本机相同的页面，无需远程桌面。**默认关闭。**
- **口令登录** —— 一切访问都需要口令。开启时自动生成随机口令，只保存哈希。连续输错会越来越慢，
  修改口令会让所有人退出登录。
- **藏在随机入口路径之后** —— 路径不对时，看起来就是页面不存在（404）。只有您允许的网络内的电脑才能连上。
- **优先 HTTPS** —— 证书覆盖服务器的网络地址；每台设备在 `/lan/ca` 页面信任一次即可。也可以选择明文 HTTP，会有提示。
- **`quantctl` 命令** —— 在服务器上用 `quantctl info`、`remote on|off`、`passwd`、`port`、`scheme`、`entrance`、
  `bind`、`trust-ca`、`doctor`、`log` 查看和修改一切设置。
  快速上手：运行 `quantctl remote on`，重启应用，再用 `quantctl info` 查看访问地址，`quantctl doctor` 自检。
- **在另一台电脑上完成 Schwab 授权** —— 新增页面（`/lan/finish-auth`），粘贴您登录 Schwab 后浏览器显示的地址即可，
  整个授权过程都可以远程完成。
- **默认更安全** —— 除非您开启远程面板，应用绝不会监听网络。如果设置文件把 `Urls` 写成了非本机地址，
  应用现在只在本机监听并在日志中说明（那种配置没有任何登录保护）。Schwab 回调地址现在固定取 `Urls` 的**第一个**地址。

---

## [1.8.4] - 2026-10-03

### English

**New: built-in read-only MCP server for AI clients.**

- **Read-only MCP (Model Context Protocol) server** — Claude Code, Cursor and other AI clients can read
  your Schwab account, positions, orders, quotes, option chains and rebalance suggestions. **It cannot place,
  change or cancel any order.**
- **Off by default** — turn it on with `Mcp:Enabled` in your settings file. Setup guide:
  https://www.alpha-wealth-lab.com/mcp.html
- **Protected by a token** — a random token is created the first time you turn it on; clients must send it.
  It only accepts connections from your own computer.
- **10 tools**: `get_account`, `get_balances`, `get_positions`, `get_orders`, `get_order`, `get_quote`,
  `get_quotes`, `get_price_history`, `get_option_chain`, `calculate_rebalance`.
- **Privacy note** — everything a tool returns is sent to the AI service your client uses.

### 中文

**新增：内置只读 MCP 服务，供 AI 客户端使用。**

- **只读 MCP（Model Context Protocol）服务** —— Claude Code、Cursor 等 AI 客户端可读取您的 Schwab 账户、
  持仓、订单、报价、期权链与再平衡建议。**无法下单、改单或撤单。**
- **默认关闭** —— 在设置文件中通过 `Mcp:Enabled` 开启。使用说明：
  https://www.alpha-wealth-lab.com/mcp.html
- **Token 保护** —— 首次开启时自动生成随机 token，客户端必须携带；仅接受来自您本机的连接。
- **10 个工具**：`get_account`、`get_balances`、`get_positions`、`get_orders`、`get_order`、`get_quote`、
  `get_quotes`、`get_price_history`、`get_option_chain`、`calculate_rebalance`。
- **隐私提示** —— 工具返回的所有内容都会发送给您的客户端所使用的 AI 服务。

---

## [1.8.3] - 2026-09-30

### English

**Fixes: fresh Linux installs now show "Secure" in the browser out of the box.**

- **Correct NSS trust flag (the root cause of "not secure").** The local CA was imported into the
  user's NSS database with the `C,,` trust flag, which does **not** mark a certificate as trusted for
  TLS. Chrome/Chromium therefore still showed `https://127.0.0.1:8443` as "not secure" on first launch.
  The CA is now imported with the `CT,CT,CT` flag, so a new install is trusted for SSL immediately.
- **PEM export for `certutil`.** The CA is now converted to PEM before the NSS import, which keeps the
  stored NSS entry valid (this also works around builds whose `quant-ca.cer` is DER).

### 中文

**修复：全新 Linux 安装开箱即显示"安全"。**

- **正确的 NSS 信任标志（"不安全"的根因）。** 之前本地 CA 以 `C,,` 标志导入用户 NSS 库，该标志并未
  将证书标记为"受信任的 TLS 根"，导致 Chrome/Chromium 首次访问 `https://127.0.0.1:8443` 仍显示
  "不安全"。现在以 `CT,CT,CT` 标志导入，新安装立即被信任。
- **PEM 导出。** 导入 NSS 前先将 CA 转为 PEM，确保 NSS 条目有效（同时兼容 `quant-ca.cer` 为 DER 的构建）。

---

## [1.8.2] - 2026-09-29

### English

**Makes the local-CA HTTPS trust step fully automatic on Linux — no commands to type.**

- **Automatic browser trust on Linux.** The local CA is now imported directly into the current
  user's NSS certificate database (`~/.pki/nssdb`), which Chrome/Chromium read on Linux — no `sudo`
  required, since it is a per-user store. Previously this required manually running `certutil`
  commands copied from the startup log.
- **Self-healing retry.** If the one dependency this needs (`certutil`, from the `libnss3-tools`
  package) isn't installed yet, the app now retries automatic trust on every subsequent startup
  instead of only once at CA creation time — installing the package and restarting is enough, with
  no need to regenerate the certificate or reconfigure anything.
- **Correct SSL trust flag + valid NSS entry.** The CA is now imported into NSS with the `CT,CT,CT`
  (SSL-trusted) flag and fed as a PEM export. The previous `C,,` flag stored the CA in NSS without
  marking it as trusted for TLS, so Chrome still showed `https://127.0.0.1:8443` as "not secure" on
  first launch; the new flag makes a fresh install show "Secure" out of the box.

### 中文

**让 Linux 上本地 CA 的 HTTPS 信任步骤完全自动化——不再需要手敲命令。**

- **Linux 上自动信任浏览器。** 本地 CA 现在会直接导入当前用户的 NSS 证书数据库
  （`~/.pki/nssdb`），Chrome/Chromium 在 Linux 上正是读取该库——因为这是用户级存储，不需要
  `sudo`。此前需要手动复制启动日志里的 `certutil` 命令来执行。
- **自愈式重试。** 如果所需的唯一依赖（`certutil`，来自 `libnss3-tools` 包）尚未安装，应用现在
  会在此后每次启动时重试自动信任，而不仅仅是在 CA 创建那一刻尝试一次——只需安装该依赖包并重启
  应用即可，无需重新生成证书或重新配置。
- **正确的 SSL 信任标志 + 有效的 NSS 条目。** 本地 CA 现在以 `CT,CT,CT`（受信任 TLS）标志导入，
  并以 PEM 传入 `certutil`（它只接受 PEM）。旧的 `C,,` 标志只把 CA 存入 NSS 却未标记为受信任，
  导致 Chrome 首次访问 `https://127.0.0.1:8443` 仍显示"不安全"；新标志让全新安装直接显示"安全"。

---
## [1.8.1] - 2026-09-28

### English

**Fixes the "not private" HTTPS warning and the update-check failure from v1.8.0.**

- **Local CA + CA-signed HTTPS certificate.** Replaces the plain self-signed certificate with a
  local Certificate Authority that is trusted once (a one-time step per machine) and then signs the
  server certificate. If you later change `Urls` or the Schwab OAuth redirect URI to a different
  host, the server certificate is automatically re-issued and signed by the same, already-trusted
  CA — no second trust step required. If CA generation fails for any reason, the app falls back to
  the old plain self-signed certificate so HTTPS never blocks startup.
- **HTTP to HTTPS redirect.** The app now recognizes `X-Forwarded-Proto` from reverse proxies
  (nginx, Apache) and forces any plain HTTP request to redirect to HTTPS, fixing the case where the
  address bar kept showing `http://` even when the app was configured for `https://`.
- **Fixed: "The update check failed" on self-contained builds.** Self-contained Release builds trim
  unused reflection metadata, which broke JSON parsing of the GitHub Releases API response and made
  the `/update` page always report a failed check. Update checking now works correctly in the
  published Setup.exe / AppImage builds.

### 中文

**修复 v1.8.0 中的 HTTPS"不安全"警告与更新检查失败问题。**

- **本地 CA + CA 签名 HTTPS 证书。** 用本地证书颁发机构（CA）替换纯自签名证书；CA 只需在每台
  机器上信任一次（一次性操作），之后由它签发服务器证书。之后如果修改 `Urls` 或 Schwab OAuth
  重定向地址到不同的主机，服务器证书会自动用同一个已被信任的 CA 重新签发，不需要第二次信任
  操作。如果 CA 生成因任何原因失败，会自动降级为旧的纯自签名证书，确保 HTTPS 问题不会阻塞启动。
- **HTTP 自动跳转 HTTPS。** 应用现在能识别反向代理（nginx、Apache）传来的 `X-Forwarded-Proto`
  头，并强制把普通 HTTP 请求重定向到 HTTPS，修复了即使配置了 `https://` 地址栏仍然显示
  `http://` 的问题。
- **修复：self-contained 构建中"The update check failed"。** self-contained 的 Release 构建会
  裁剪未使用的反射元数据，导致解析 GitHub Releases API 响应的 JSON 时失败，`/update` 页面因此
  总是报告检查失败。现在已发布的 Setup.exe / AppImage 构建中更新检查功能正常工作。

---

## [1.8.0] - 2026-09-27

### English

**One-click upgrade — update from inside the app; configuration and strategy data are never lost.**

- **In-app updater (Velopack).** The app checks GitHub Releases on startup and every 24 hours.
  When a new version is available a banner appears at the top of the page. One click downloads,
  applies and restarts into the new version on Windows, macOS and Linux.
- **`/update` page.** Shows the installed version, the latest version, the release notes from this
  changelog and the download progress. Buttons: `Check now`, `Download`, `Upgrade & Restart` and
  `Upgrade after market close`.
- **Dismissible banner.** Dismissing a version hides it until a newer version appears;
  the dismissal is stored in your user settings, not in the program folder.
- **Trading safety gate.** An upgrade is refused while a rebalance is running, while orders are
  still working/pending/queued, or — with automatic mode enabled — during US market hours
  (09:30–16:00 ET on weekdays). Refusals explain the reason instead of silently doing nothing.
  `Upgrade after market close` retries the gate after 16:30 ET.
- **Pre-upgrade snapshot.** `config/` and `Data/` are copied to `backups/{UTC}-{old version}/`
  before every upgrade; the five most recent snapshots are kept. Tokens and credentials are never
  copied. If the post-upgrade data migration fails, the latest snapshot is restored automatically.
- **Post-upgrade feedback.** After a restart into a new version the app opens the dashboard and
  shows `Updated to vX.Y.Z`.
- **Degraded mode for zip installations.** If the app was not installed through the Velopack
  installer, it only compares versions against the public release repository and offers the installer
  download. Nothing on your disk is replaced automatically; after one manual installer run, every
  later upgrade is a single click.
- **Public release repository.** Releases are published to a separate, **public** repository
  (`memoryfraction/Quant.Infra.Net.Pro-Public`) that holds nothing but release assets, so checking for
  an update and downloading the installer need no account at all. The source repository stays private.
  Its address is the `Update:ReleaseRepoUrl` setting, and all four consumers read it: the Velopack
  feed, the degraded-mode release check, the installer download link and the `How to upgrade` link.
- **`How to upgrade` link.** The update banner and the `/update` page link to the upgrade
  instructions published with the release repository.
- **Version single source.** This changelog is embedded into the app and drives the version history
  page; the git tag drives the build version. No more hand-copied version numbers.
- **Release pipeline.** `CS 架构/scripts/build-release.ps1` builds self-contained packages for every
  platform locally; `.github/workflows/release.yml` publishes them to GitHub Releases with
  full and delta (incremental) packages.
- **`Upgrade anyway` (when open orders cannot be verified).** If the open-orders check cannot be
  completed at all (for example Schwab is not connected, or the authorization has expired), the
  `/update` page offers an `Upgrade anyway (I have no open orders)` button with a warning. The
  confirmation only bypasses the "orders could not be verified" refusal; it can never bypass a
  running rebalance, clearly in-flight orders, or the market-hours rule, and `Upgrade after
  market close` never uses this bypass.
- **MVP scope.** Windows is the fully supported one-click upgrade target; Linux ships an
  AppImage (packaged, not end-to-end verified in this cycle); macOS is not published in this
  cycle — its CI job runs only on a manual `workflow_dispatch` with the `include_macos` input
  set to true.

### 中文

**一键升级 —— 在应用内完成升级，配置与策略数据永不丢失。**

- **应用内升级器（Velopack）。** 启动后检查一次 GitHub Releases，之后每 24 小时检查一次。
  有新版本时页面顶部出现提示横幅。Windows、macOS、Linux 上点一下即完成下载、替换与重启。
- **`/update` 页面。** 显示当前版本、最新版本、来自本变更记录的更新说明与下载进度。
  按钮包括 `Check now`、`Download`、`Upgrade & Restart`、`Upgrade after market close`。
- **可关闭的横幅。** 关闭某个版本后不再提示该版本，出现更新版本时仍会提示；
  关闭状态写入用户设置，而不是程序目录。
- **交易安全闸门。** 再平衡执行中、存在未完成订单、或开启自动模式且处于美股交易时段
  （美东工作日 09:30–16:00）时拒绝升级，并明确说明原因。`Upgrade after market close`
  会在美东 16:30 之后重新检查闸门再执行。
- **升级前快照。** 每次升级前把 `config/` 与 `Data/` 复制到 `backups/{UTC}-{旧版本}/`，
  只保留最近 5 份；不复制 token 与凭据。升级后的数据迁移若失败，会自动从最近快照恢复。
- **升级后反馈。** 重启进入新版本后自动打开 Dashboard 并显示 `Updated to vX.Y.Z`。
- **zip 安装的降级模式。** 如果不是通过 Velopack 安装器安装的，只与公开发布仓库比对版本
  并提供安装器下载链接，不会自动替换磁盘上的任何内容。手动装一次安装器后，之后每次升级都只需一点。
- **公开发布仓库。** 发布投递到独立的**公开**仓库（`memoryfraction/Quant.Infra.Net.Pro-Public`），
  只承载发布资产，因此检查更新与下载安装器都无需任何账号；源代码仓库保持私有。
  该地址即 `Update:ReleaseRepoUrl` 设置，四个消费者全部读取它：Velopack 发布源、
  降级模式的版本检查、安装器下载链接、`How to upgrade` 链接。
- **`How to upgrade` 链接。** 升级横幅与 `/update` 页面均链接到随发布仓库一同发布的升级说明。
- **版本单一来源。** 本变更记录嵌入程序并驱动版本历史页；git tag 驱动构建版本号，不再手工复制版本号。
- **发布流水线。** `CS 架构/scripts/build-release.ps1` 在本地构建各平台 self-contained 包；
  `.github/workflows/release.yml` 把全量包与增量包发布到 GitHub Releases。
- **`Upgrade anyway`（无法核实在途订单）。** 当在途订单查询完全无法核实时（例如 Schwab 未连接、
  或授权已过期），`/update` 页面会提供 `Upgrade anyway (I have no open orders)` 按钮并附警示语。
  该确认只绕过“无法核实”这一种拒绝 - 再平衡执行中、明确在途的订单、或交易时段规则永远无法绕过，
  `Upgrade after market close` 也不使用该绕过。
- **MVP 范围。** Windows 为完整支持的一键升级目标；Linux 仅交付 AppImage（只打包，本周期未做端到端
  验证）；macOS 本周期不发布，其 CI 任务仅在手动 `workflow_dispatch` 且 `include_macos` 输入为 true 时运行。

---

## [1.7.0] - 2026-09-27

### English

**Program / data separation — configuration and strategy data now survive every upgrade.**

This release is the prerequisite stage of the one-click upgrade plan. It does not yet add an
in-application updater; it removes the root cause of "my settings disappeared after upgrading".

- **User data moved out of the program folder.** Configuration, strategy data, trigger settings,
  execution mode, logs and the HTTPS certificate now live under
  `%APPDATA%\Quant.Infra.Net.Pro\` (`~/.config/Quant.Infra.Net.Pro/` on macOS/Linux).
  Overwriting the program folder — by zip or by installer — can no longer destroy user data.
- **Configuration layering.** Shipped `appsettings.json` (read-only defaults) →
  `config/appsettings.user.json` (your overrides) → environment variables → User Secrets.
  New settings introduced by a release arrive automatically as defaults; your edited values stay yours.
- **One-time automatic migration.** On first launch the app moves License Key, Urls, RedirectUri,
  advanced overrides, the trigger configuration, the HTTPS certificate and the EULA acceptance
  record out of the old install folder. Idempotent: running it again changes nothing, and it never
  overwrites an existing file.
- **Fixed trigger configuration bug.** The rebalance trigger settings were written into the program
  folder but read from the user data folder, so saved trigger settings were never read back and were
  lost on upgrade. Read and write now use the same path.
- **Manual / automatic execution mode is persisted.** Previously it reset to Manual every time the
  Rebalance page was opened.
- **SchemaVersion + migration pipeline.** User data files carry a schema version and are upgraded by
  an ordered `IUserDataMigration` pipeline. Deserialization tolerates unknown fields, so installing
  an older version back remains possible.
- **Export / import all settings** (Settings page). Download a zip with your configuration and
  strategy data for machine migration or manual backup. Schwab credentials are excluded by default
  and require an explicit opt-in; authorization tokens are never exported.
- **Data root override.** The `QUANT_INFRA_DATA_DIR` environment variable overrides the user data
  root, which is required for Linux service accounts running as root or under systemd.
- **Documentation corrected.** The deployment guide claimed configuration files were preserved
  across an upgrade; that was not true before this release. README now documents where data lives.

Chinese notes for this release are in the section below the English one.

### 中文

**程序与数据分离 —— 配置和策略数据从此不会被升级破坏。**

本版本是一键升级方案的前置阶段。它尚未引入应用内升级器，而是先消除"升级后设置丢失"的根因。

- **用户数据移出程序目录。** 配置、策略数据、触发配置、执行模式、日志与 HTTPS 证书统一迁移到
  `%APPDATA%\Quant.Infra.Net.Pro\`（macOS/Linux 为 `~/.config/Quant.Infra.Net.Pro/`）。
  无论用 zip 覆盖还是安装器升级程序目录，都不会再破坏用户数据。
- **配置分层。** 随程序发布的 `appsettings.json`（只读默认值）→ `config/appsettings.user.json`
  （你的覆盖项）→ 环境变量 → User Secrets。新版本新增的配置项自动获得默认值，你改过的值保持不动。
- **一次性自动迁移。** 首次启动时自动把旧目录里的 License Key、Urls、RedirectUri、高级覆盖项、
  触发配置、HTTPS 证书与 EULA 接受记录搬到用户数据区。幂等：重复运行不会产生变化，且绝不覆盖已存在的文件。
- **修复触发配置 Bug。** 再平衡触发配置过去写入程序目录、却从用户数据区读取，导致保存的设置永远读不回来、
  且升级即丢失。现读写使用同一路径。
- **手动/自动执行模式持久化。** 此前每次打开再平衡页面都会重置为手动模式。
- **SchemaVersion 与迁移管线。** 用户数据文件带架构版本，由有序的 `IUserDataMigration` 管线升级。
  反序列化容忍未知字段，因此仍可装回旧版本。
- **导出/导入全部设置**（Settings 页面）。下载包含配置与策略数据的 zip，用于换机器或人工备份。
  Schwab 凭据默认不包含、需明确勾选；授权 token 永不导出。
- **数据根目录可覆盖。** 环境变量 `QUANT_INFRA_DATA_DIR` 可覆盖用户数据根目录，
  这是 Linux 以 root/systemd 运行服务账号所必需的。
- **文档修正。** 部署指南过去声称"配置文件会保留"，在本版本之前该说法并不成立。
  README 现已补充数据存放位置说明。

---

## [1.6.3] - 2026-07-11

Privacy mask for financial data: eye-toggle on Dashboard and Rebalance pages. Sensitive amounts and
account numbers masked as `****` by default.
财务数据隐私脱敏：Dashboard 与 Rebalance 页面增加眼睛图标开关，敏感金额与账户号码默认遮盖为 `****`。

---

Earlier releases (1.6.2 and before) are documented in `README.md`.
更早的版本（1.6.2 及以前）记录在 `README.md` 中。
