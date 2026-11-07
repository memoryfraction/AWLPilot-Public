# AWLPilot

> **Keep your own ETFs on target, automatically, in the Schwab account you already have.**
> Set your target weights once (say 60% VTI / 40% BND). AWLPilot watches the drift and brings you back. It runs on your own computer: your money, your password and your trading data never leave your machine.

[Website & pricing](https://www.alpha-wealth-lab.com/pricing/) · [Download](https://github.com/memoryfraction/AWLPilot-Public/releases/latest) · [Quick-start guide](Quick-start%20Guide.md) · [Ask an AI agent to set it up](AGENTS.md) · [中文](#中文)

> **Naming note.** The product is now called **AWLPilot** (formerly *Quant.Infra.Net.Pro*). The installer, the program and the data folder still carry the old name for now, so you will see `QuantInfraNetPro` and `Quant.Infra.Net.Pro` below. Nothing else changes.

# English

## Why it exists

Schwab only auto-rebalances inside *Intelligent Portfolios*, with Schwab's funds and a forced cash allocation. If you hold your own ETFs in a normal account, rebalancing means doing the arithmetic and placing the trades yourself. Doing it with scripts through Schwab's API breaks every 7 days, when the refresh token expires.

AWLPilot closes that gap: **your own mix, your own account, rebalanced automatically.**

## What it does

| | |
|---|---|
| **Rebalance by your rules** | Set target weights; see current drift and the exact trade list; run it manually, or let it run automatically (Pro). Safety guardrails block automatic runs while orders are in flight. |
| **Scheduled check (optional, off by default)** | When and how to rebalance is your strategy, so AWLPilot does nothing on its own unless you turn this on. Turned on, it checks your saved trigger once each weekday at 10:30–11:30 ET: in automatic mode with Pro it places the orders, otherwise it emails you the trade list. Turn on: add `"Rebalance": { "Schedule": { "Enabled": true } }` to `config/appsettings.user.json` and restart. |
| **Stays connected** | Handles Schwab's 30-minute access token and 7-day refresh token for you, including automatic re-login. If Schwab asks for MFA it stops and tells you; MFA is never bypassed. |
| **Local first** | Runs on your own PC or server. Credentials, tokens, trades and logs stay on your machine. Sensitive amounts are masked by default. |
| **Works with your AI agent** | Optional read-only MCP server so Claude Code, Cursor and others can read your account and rebalance suggestions. It has no tool that places or cancels orders. See [MCP.md](MCP.md). |
| **Also a developer API** | Accounts, positions, quotes, option chains, price history and order history over a local REST API with Swagger. |

## Get started in 10 minutes

1. **Download** from [Releases](https://github.com/memoryfraction/AWLPilot-Public/releases/latest): `QuantInfraNetPro.Web-win-x64-Setup.exe` (Windows) or `QuantInfraNetPro.Web-linux-x64.AppImage` (Linux). No separate .NET install needed. There is no macOS build at the moment.
2. **Install and run it**, then open **https://127.0.0.1:8443**. The browser warns about the certificate on first launch: that is expected for a local certificate, follow the on-screen note to trust it once.
3. **Start the free trial** on the `/start` page: verify your email, no card needed.
4. **Connect your Schwab account** on `/settings` with your own Schwab Trader API App Key, App Secret and account number. You need a Schwab brokerage account and an approved Individual Developer app; registration needs a +1 phone number and approval takes a few business days. Step by step: [Charles Schwab Developer Registration Guide](Charles%20Schwab%20Developer%20Registration%20Guide.md).
5. **Set your target weights** on `/Rebalance` and check the trade list before anything runs.

**Running it on another computer (a home server) and using it from your PC?** Follow [Two-Computer-Setup.md](Two-Computer-Setup.md). Short version: keep the Schwab callback URL as `https://127.0.0.1:8443/`; open the address printed by `quantctl info` on the server; when Schwab login ends on a page that will not open, copy that whole address into `/lan/finish-auth`. Schwab requires this login again every 7 days.

Stuck? Let your AI agent do it: give [AGENTS.md](AGENTS.md) to Claude Code or Cursor and say "install AWLPilot for me".

## Price

Free to download and try; the in-app trial includes every feature. Pro is **$49/month** or **$490/year**, or **$490 for one year** by Alipay / WeChat Pay. A **Family** plan ($9,800/year) adds one-to-one technical service. Buy from the [pricing page](https://www.alpha-wealth-lab.com/pricing/); your license key arrives by email.

## Support

- Bugs and questions: [GitHub Issues](https://github.com/memoryfraction/AWLPilot-Public/issues) or alphawealthlab@outlook.com. No promised response time on Pro; Family has priority replies.
- Install on one computer, use from another (step by step, 两台电脑详细步骤): [Two-Computer-Setup.md](Two-Computer-Setup.md). Full Linux server walkthrough: [LAN-Server-Setup.md](LAN-Server-Setup.md). Telegram: https://t.me/+VPy-VLis8gVmYWM1
- Release notes: [CHANGELOG.md](CHANGELOG.md) and the [website changelog](https://www.alpha-wealth-lab.com/changelog).

## What AWLPilot is not

Software you run yourself, not a robo-adviser. We do not hold your money, open accounts, place trades on anyone's behalf, or give investment advice. You connect your own Schwab account with your own API credentials and you set the rules. Read the [Risk Disclaimer](CHARLES_SCHWAB_DISCLAIMER.md) and the [Data Privacy Statement](DATA_PRIVACY.md). Not affiliated with or endorsed by Charles Schwab.

## Upgrading

**Install once, then upgrade from inside the app.** From v1.8.0 the app checks for a new version two
minutes after it starts and every 24 hours after that. When one is found, a banner appears at the top of
every page and `/update` shows the installed version, the latest version, the release notes and a single
`Upgrade & Restart` button — no reinstall, no re-download of a ZIP.

- Your configuration and strategy data live in `%APPDATA%\Quant.Infra.Net.Pro`, **outside** the program
  folder, so an upgrade never overwrites them.
- A snapshot of `config/` and `Data/` is taken before every upgrade and the five most recent snapshots
  are kept under `backups/`.
- The upgrade is **refused while trading is in flight** — a running rebalance, working or queued orders,
  or US market hours (09:30–16:00 ET on weekdays) with automatic mode on. The reason is always shown.
  `Upgrade after market close` retries the check after 16:30 ET.
- ZIP installs and development builds cannot replace themselves while running; they show a download link
  instead and become one-click installs as soon as you install the installer once.

## Where your data lives

Since **v1.7.0** the program folder and your data are strictly separated. An upgrade — whether it is
an automatic update or a manual zip overwrite — only replaces the program folder. Your configuration
and strategy data are never touched.

| Data | Location |
|------|----------|
| Configuration overrides (License Key, Email, Urls, RedirectUri, advanced settings) | `%APPDATA%\Quant.Infra.Net.Pro\config\appsettings.user.json` |
| Target portfolio, rebalance ratio, drift thresholds, email notification (**the strategy itself**) | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-settings.json` |
| Rebalance trigger configuration | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-trigger.json` |
| Manual / Automatic execution mode | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-mode.json` |
| HTTPS certificate (`quant-infra.pfx` / `.cer`) | `%APPDATA%\Quant.Infra.Net.Pro\certs\` |
| Logs (rolling daily) | `%APPDATA%\Quant.Infra.Net.Pro\logs\` |
| Schwab authorization tokens | `%APPDATA%\Quant.Infra.Net.Pro\tokens.dat` (+ `tokens.key`) |
| EULA acceptance record | `%APPDATA%\Quant.Infra.Net.Pro\eula-acceptance.json` |
| Schwab API credentials (Charles Key / Secret / login) | `%APPDATA%\Microsoft\UserSecrets\schwab-web-app-secrets\secrets.json` |

Configuration is layered, later sources win: shipped `appsettings.defaults.json` (read-only defaults) →
`config\appsettings.user.json` (your overrides) → environment variables → User Secrets. Settings added
by a new release therefore arrive automatically as defaults, while the values you edited stay yours.

**Prefer the old way?** Copying an `appsettings.json` into the program folder still works on every OS: on the next
start its License, Urls, RedirectUri and advanced values are applied (Schwab credentials go to User Secrets if
not already set), and the file is renamed to `appsettings.json.migrated`. The Settings page shows the data folder
with an **Open folder** button, and the console prints the settings file path at startup.

On macOS and Linux the root is `~/.config/Quant.Infra.Net.Pro/`. On Linux the environment variable
`QUANT_INFRA_DATA_DIR` overrides the root — required when running as root or under systemd so that
data does not land in `/root/.config`.

**Upgrading from a version before 1.7.0**: extract the new version **over the old folder** (releases no longer
ship `appsettings.json`, so your old one stays in place). On first launch the app migrates the old
data automatically — License Key, Urls, RedirectUri, advanced overrides, the trigger configuration,
the HTTPS certificate and the EULA record. Your Schwab authorization stays valid, because tokens were
already stored outside the program folder and never move. The Settings page also provides
**Export all settings** / **Import settings** for machine migration or a manual safety copy
(Schwab credentials are excluded unless you explicitly opt in; authorization tokens are never exported).
If you extracted the new version into a **different** folder, use **Settings → Import from a previous
version folder** and point it at the old folder.

---

# 中文

## 一句话

> **按你自己定的比例，自动再平衡你现有的嘉信账户。**
> 设一次目标权重（比如 60% VTI / 40% BND），AWLPilot 检查偏离并调回去。软件跑在你自己的电脑上：钱、密码、交易数据都不出你的电脑。

> **名称说明：** 产品现在叫 **AWLPilot**（原名 *Quant.Infra.Net.Pro*）。安装包、程序和数据文件夹暂时仍沿用旧名，看到 `QuantInfraNetPro`、`Quant.Infra.Net.Pro` 属正常，其他没有变化。

## 为什么做它

嘉信只在 *Intelligent Portfolios* 里自动再平衡，而且用它选的基金、强制留现金。你在普通账户里持有自己的 ETF，就只能自己算、自己下单；用脚本调 Schwab API，每 7 天 refresh token 一过期就断。AWLPilot 补的就是这个空档：**你自己的组合、你自己的账户，自动再平衡。**

## 能做什么

| | |
|---|---|
| **按你的规则再平衡** | 设目标权重；看到当前偏离和要下的每一笔单；手动执行，或（Pro）自动执行。有订单在途时，安全护栏会拦住自动执行。 |
| **定时检查（可选，默认关闭）** | 何时、如何再平衡是你自己的策略，没打开时 AWLPilot 不会自己做任何事。打开后，每个工作日美东 10:30–11:30 按你保存的触发条件检查一次：自动模式 + Pro 直接下单，否则把动作单发到你的邮箱。打开方法：在 `config/appsettings.user.json` 里加 `"Rebalance": { "Schedule": { "Enabled": true } }`，然后重启。 |
| **一直保持连接** | 自动处理 Schwab 30 分钟的 access token 和 7 天的 refresh token，包括自动重新登录。遇到 MFA 会停下来提醒你，不会绕过。 |
| **数据留在本机** | 跑在你自己的电脑或服务器上。凭据、Token、交易和日志都在本机，敏感金额默认遮盖。 |
| **配合你的 AI Agent** | 可选的只读 MCP 服务，Claude Code、Cursor 等可读取账户和再平衡建议；没有任何下单或撤单工具。见 [MCP.md](MCP.md)。 |
| **也是开发者 API** | 本地 REST API（带 Swagger）：账户、持仓、行情、期权链、历史行情、订单历史。 |

## 10 分钟上手

1. 到 [Releases](https://github.com/memoryfraction/AWLPilot-Public/releases/latest) 下载：`QuantInfraNetPro.Web-win-x64-Setup.exe`（Windows）或 `QuantInfraNetPro.Web-linux-x64.AppImage`（Linux）。无需另装 .NET；目前没有 macOS 版本。
2. 安装并运行，浏览器打开 **https://127.0.0.1:8443**。首次启动会提示证书不受信任，这是本地证书的正常现象，按页面提示信任一次即可。
3. 在 `/start` 页面开始免费试用：邮箱验证即可，无需绑卡。
4. 在 `/settings` 填入你自己的 Schwab Trader API 的 App Key、App Secret 和账号。需要嘉信账户和已批准的个人开发者应用；注册需要 +1 电话号码，审核要几个工作日。详见 [Charles Schwab 开发者注册指南](Charles%20Schwab%20Developer%20Registration%20Guide.md)。
5. 在 `/Rebalance` 设目标权重，先检查动作单，再决定执行。

**装在另一台电脑（家里服务器）上、在自己电脑上用？** 按 [Two-Computer-Setup.md](Two-Computer-Setup.md) 做。要点：嘉信开发者应用的 Callback URL 保持 `https://127.0.0.1:8443/` 不变；在服务器上运行 `quantctl info`，打开它打印的地址；登录嘉信后跳到一个打不开的页面时，把整条地址复制到 `/lan/finish-auth`。嘉信要求每 7 天重新登录一次。

卡住了？让你的 AI Agent 来：把 [AGENTS.md](AGENTS.md) 给 Claude Code 或 Cursor，说“帮我安装 AWLPilot”。

## 价格

免费下载、免费试用，试用期内全部功能可用。Pro **$49/月** 或 **$490/年**，或用支付宝/微信一次性 **$490 买一年**；Family（$9,800/年）另含一对一技术服务。在[定价页](https://www.alpha-wealth-lab.com/pricing/)购买，License 发到你的邮箱。

## 支持

问题和 Bug：[GitHub Issues](https://github.com/memoryfraction/AWLPilot-Public/issues) 或 alphawealthlab@outlook.com；Pro 不承诺响应时间，Family 优先回复。更新记录见 [CHANGELOG.md](CHANGELOG.md)。

## AWLPilot 不是什么

它是你自己运行的软件，不是机器人投顾。我们不托管资金、不代客开户、不代客下单、不提供投资建议。你用自己的嘉信账户和 API 凭证连接，规则由你设定。请阅读[风险免责声明](CHARLES_SCHWAB_DISCLAIMER.md)和[数据隐私声明](DATA_PRIVACY.md)。与嘉信无关联，也未获其认可。

## 升级方式

**装一次安装器，之后在应用内一键升级。** 从 v1.8.0 起，应用在启动 2 分钟后检查一次新版本，之后每 24 小时检查一次。
发现新版本时，页面顶部会出现提示横幅，`/update` 页面显示已安装版本、最新版本、更新说明，以及一个
`Upgrade & Restart` 按钮——无需重装，也无需重新下载 ZIP。

- 配置与策略数据存放在 `%APPDATA%\Quant.Infra.Net.Pro`，**在程序目录之外**，升级绝不会覆盖它们。
- 每次升级前都会把 `config/` 与 `Data/` 快照到 `backups/`，保留最近 5 份。
- **交易进行中会拒绝升级**——再平衡正在执行、存在未完成订单，或自动模式下处于美股交易时段
  （工作日 09:30–16:00 ET）。拒绝时会说明原因。`Upgrade after market close` 会在 16:30 ET 后重试。
- ZIP 安装与开发构建在运行期间无法自我替换，它们显示下载链接；只要安装一次安装器，就会变成一键升级。

## 数据存放位置

自 **v1.7.0** 起，程序目录与你的数据严格分离。升级（无论是自动更新还是手动覆盖 zip）只替换程序目录，
**配置与策略数据永远不会被动到**。

| 数据 | 位置 |
|------|------|
| 配置覆盖项（License Key、邮箱、Urls、RedirectUri、高级设置） | `%APPDATA%\Quant.Infra.Net.Pro\config\appsettings.user.json` |
| 目标持仓、再平衡比例、漂移阈值、邮件通知（**策略本体**） | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-settings.json` |
| 再平衡触发配置 | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-trigger.json` |
| 手动/自动执行模式 | `%APPDATA%\Quant.Infra.Net.Pro\Data\rebalance-mode.json` |
| HTTPS 证书（`quant-infra.pfx` / `.cer`） | `%APPDATA%\Quant.Infra.Net.Pro\certs\` |
| 日志（按天滚动） | `%APPDATA%\Quant.Infra.Net.Pro\logs\` |
| Schwab 授权 token | `%APPDATA%\Quant.Infra.Net.Pro\tokens.dat`（含 `tokens.key`） |
| EULA 接受记录 | `%APPDATA%\Quant.Infra.Net.Pro\eula-acceptance.json` |
| Schwab API 凭据（Charles Key / Secret / 登录名） | `%APPDATA%\Microsoft\UserSecrets\schwab-web-app-secrets\secrets.json` |

配置为分层结构，后加载的覆盖先加载的：随程序发布的 `appsettings.defaults.json`（只读默认值）→
`config\appsettings.user.json`（你的覆盖项）→ 环境变量 → User Secrets。
因此新版本新增的配置项自动以默认值补齐，而你改过的值保持不动。

**习惯老办法？** 在任何系统上，把 `appsettings.json` 复制到程序目录依然有效：下次启动时会应用其中的 License、Urls、
RedirectUri 与高级设置（Schwab 凭据若尚未设置则写入 User Secrets），随后文件改名为 `appsettings.json.migrated`。
Settings 页显示数据目录并提供 **Open folder** 按钮，启动时控制台也会打印设置文件路径。

macOS 与 Linux 的根目录为 `~/.config/Quant.Infra.Net.Pro/`。Linux 上可用环境变量
`QUANT_INFRA_DATA_DIR` 覆盖根目录——以 root 或 systemd 运行时必须设置，否则数据会落到 `/root/.config`。

**从 1.7.0 之前的版本升级**：把新版本**解压覆盖到旧目录**即可（发布包已不含 `appsettings.json`，旧文件会原样保留）。首次启动会自动迁移旧数据——License Key、Urls、
RedirectUri、高级覆盖项、触发配置、HTTPS 证书与 EULA 记录。Schwab 授权保持有效，因为 token 本来
就存放在程序目录之外，从不移动。Settings 页面还提供 **Export all settings** / **Import settings**，
用于换机器或人工兜底备份（Schwab 凭据默认不含、需明确勾选；授权 token 永不导出）。
如果把新版本解压到了**另一个**目录，请在 **Settings → Import from a previous version folder** 中填写旧目录路径。
