# Two computers: install on one, use from the other / 两台电脑：装在一台，在另一台上用

[English](#english) · [中文](#中文)

> Placeholders used below / 下文占位符：
> `<SERVER-IP>` the address of the computer that runs AWLPilot (LAN or VPN address) / 运行 AWLPilot 的那台电脑的地址（局域网或虚拟网地址）·
> `<USER>` your login name on that computer / 你在那台电脑上的登录名 ·
> `<SECRET-PATH>` the random path printed by `quantctl info` / `quantctl info` 打印的随机路径。
> Nothing here contains a real address, password or token. Keep yours private.
> 本文不含任何真实地址、口令或令牌，请妥善保管你自己的。

---

## English

### The idea in one minute

- **The server** is the computer that keeps AWLPilot running (for example a small always-on Linux box). It holds your Schwab connection.
- **Your PC** is the computer you sit at. It only needs a web browser.
- By default the program listens **only on its own computer** (`127.0.0.1`). That is a safety feature, and it is why a browser on another computer cannot open it directly.
- There are two ways to reach it from your PC. Pick one:

| | Way A: SSH tunnel | Way B: remote panel |
|---|---|---|
| What you open on your PC | `https://127.0.0.1:8443` | `https://<SERVER-IP>:8443/<SECRET-PATH>` |
| Why the address looks odd | The tunnel makes your PC's port 8443 an alias for the server's. `127.0.0.1` here means "the tunnel's entrance on my PC". | It is the real server address. |
| Setup | One command on your PC. Nothing to change on the server. | A few commands on the server, plus a password. |
| Best for | One person, occasional use. **Start here.** | Several devices, or you do not want to keep a tunnel running. |
| Needs | SSH access to the server | A trusted private network (LAN, ZeroTier, Tailscale). **Never expose it to the public internet.** |

### Step 1: install on the server (once)

On the server, as your normal user (no root needed for the program itself):

```bash
mkdir -p ~/Applications
curl -fL -o ~/Applications/Quant.Infra.Net.Pro.AppImage \
  "https://github.com/memoryfraction/AWLPilot-Public/releases/latest/download/QuantInfraNetPro.Web-linux-x64.AppImage"
chmod +x ~/Applications/Quant.Infra.Net.Pro.AppImage
~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli      # creates the ~/bin/quantctl helper
```

Make it start by itself and keep running. Create `/etc/systemd/system/quant-infra-net-pro.service` (needs sudo):

```ini
[Unit]
Description=AWLPilot
After=network-online.target

[Service]
User=<USER>
ExecStart=/home/<USER>/Applications/Quant.Infra.Net.Pro.AppImage
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now quant-infra-net-pro
~/bin/quantctl status          # self-check; read the first [XX] line and its fix hint
```

Windows server: install `QuantInfraNetPro.Web-win-x64-Setup.exe` instead; the program then starts from the Start menu. On Windows run control commands as
`"%LOCALAPPDATA%\QuantInfraNetPro.Web\current\Quant.Infra.Net.Pro.Web.exe" ctl <command>`.

### Step 2, Way A: SSH tunnel (simplest)

On **your PC**, open a terminal (Windows 10/11: PowerShell; it already has `ssh`) and run:

```bash
ssh -N -L 8443:127.0.0.1:8443 <USER>@<SERVER-IP>
```

- The window will look frozen. That is normal: **leave it open** while you use AWLPilot.
- Open **`https://127.0.0.1:8443`** in your PC's browser.
- If your PC already runs its own AWLPilot on 8443, use another local port, for example `-L 9443:127.0.0.1:8443`, and open `https://127.0.0.1:9443`. **Schwab sign-in then needs the extra paste step in Step 3.**
- Stop it with Ctrl+C. Use a key instead of a password for SSH if you can (`ssh-keygen`, then `ssh-copy-id <USER>@<SERVER-IP>`).

### Step 2, Way B: remote panel (open the server's address directly)

On the **server**:

```bash
~/bin/quantctl bind --list                    # shows the server's private addresses
~/bin/quantctl remote on --bind <SERVER-IP>   # choose ONE private address; prints a one-time password
sudo systemctl restart quant-infra-net-pro
~/bin/quantctl info                           # prints the full address, including the secret path
~/bin/quantctl doctor                         # self-check and firewall advice
```

- If the server has several private addresses (for example a LAN and a VPN) the program will not guess; use `--bind` with the one your PC can reach.
- If the server's firewall is on, allow port 8443 **only from your own network**. On Ubuntu, for example: `sudo ufw allow from <YOUR-NETWORK-CIDR> to any port 8443 proto tcp`.
- Change the password any time with `~/bin/quantctl passwd`.
- On your PC open the address from `quantctl info`: `https://<SERVER-IP>:8443/<SECRET-PATH>`. A wrong path shows "404" on purpose. Treat the path and the password like a password.

### Certificate warning in the browser

The program makes its own local certificate, so the first time the browser says "Not secure". For your own computers this is expected. Click through, or install the program's certificate once: run `~/bin/quantctl trust-ca` on the server and follow its instructions on your PC.

### Step 3: first-time setup in the browser (once)

1. **Accept the EULA** (you must click this yourself).
2. **Start the trial or enter your license** on `/start` or `/settings`. A trial needs email verification.
3. **Enter your Schwab App Key, App Secret and account number** on `/settings`. You need an approved Schwab Individual Developer app first: [Developer Registration Guide](Charles%20Schwab%20Developer%20Registration%20Guide.md).
4. **Sign in to Schwab**: open `/signin` and log in with your own Schwab account.
5. After the Schwab login the browser jumps to a page that **will not open**, with an address like `https://127.0.0.1:8443/?code=...`.
   - **Way A with port 8443:** it arrives through the tunnel and finishes by itself.
   - **Way B, or any other local port:** this is normal. Copy the **whole address** from the address bar, open `/lan/finish-auth` in the panel, paste it and submit. The code expires within minutes.
6. Open `/Rebalance`, set target weights and check the trade list before anything runs.

### Everyday use and maintenance

| Task | Where |
|---|---|
| Use it | Way A: start the tunnel, open `https://127.0.0.1:8443`. Way B: open the address from `quantctl info`. |
| Schwab sign-in expires every 7 days | A banner appears on the dashboard. Click **Sign In** and repeat step 4 and 5. |
| Upgrade | Open `/update` when the US market is closed and click **Upgrade & Restart**. |
| Health check | `~/bin/quantctl status` and `~/bin/quantctl doctor` on the server |
| Logs | `~/bin/quantctl log --tail 30` |
| Free disk space | After an upgrade the previous AppImage stays next to the new one as `*.old` for rollback. Delete old copies when you no longer need them. |

### Troubleshooting

| You see | Do this |
|---|---|
| "This site can't be reached", refused to connect, on `127.0.0.1` | Way A: the tunnel window was closed. Start it again. Also check the program runs on the server: `systemctl status quant-infra-net-pro`. |
| Cannot open `<SERVER-IP>:8443` | The remote panel is off (`quantctl info`), the program was not restarted, the firewall blocks 8443, or you are not on the same network. |
| Page shows 404 | The secret path is wrong. Run `quantctl info` again. |
| Dashboard says "Authorization has expired" | Normal after 7 days or at first use. Sign in to Schwab again (step 4 and 5). |
| `ssh: Permission denied` | Wrong user name, or the server has no key or password login for you. |

---

## 中文

### 一分钟看懂

- **服务器**：一直开着、跑着 AWLPilot 的那台电脑（比如一台小型 Linux 机器），嘉信连接保存在它上面。
- **你的电脑**：你平时用的那台，只要有浏览器。
- 程序默认**只监听它自己这台电脑**（`127.0.0.1`）。这是安全设计，也是为什么另一台电脑的浏览器不能直接打开它。
- 从你的电脑连过去有两种办法，选一种：

| | 办法 A：SSH 隧道 | 办法 B：远程面板 |
|---|---|---|
| 在你的电脑上打开 | `https://127.0.0.1:8443` | `https://<SERVER-IP>:8443/<SECRET-PATH>` |
| 为什么地址看起来怪 | 隧道让你电脑的 8443 端口变成服务器 8443 的"分身"。这里的 `127.0.0.1` 指"我电脑上的隧道入口"。 | 就是服务器的真实地址。 |
| 设置 | 你的电脑上一条命令，服务器不用改。 | 服务器上几条命令，外加一个口令。 |
| 适合 | 一个人、偶尔用。**先用这个。** | 多个设备，或不想一直开着隧道。 |
| 需要 | 能 SSH 登录服务器 | 可信的内网（局域网、ZeroTier、Tailscale）。**绝不要暴露到公网。** |

### 第 1 步：在服务器上安装（一次）

在服务器上用你的普通用户（程序本身不需要 root）：

```bash
mkdir -p ~/Applications
curl -fL -o ~/Applications/Quant.Infra.Net.Pro.AppImage \
  "https://github.com/memoryfraction/AWLPilot-Public/releases/latest/download/QuantInfraNetPro.Web-linux-x64.AppImage"
chmod +x ~/Applications/Quant.Infra.Net.Pro.AppImage
~/Applications/Quant.Infra.Net.Pro.AppImage ctl install-cli      # 生成 ~/bin/quantctl 工具
```

让它开机自动启动并常驻。创建 `/etc/systemd/system/quant-infra-net-pro.service`（需要 sudo）：

```ini
[Unit]
Description=AWLPilot
After=network-online.target

[Service]
User=<USER>
ExecStart=/home/<USER>/Applications/Quant.Infra.Net.Pro.AppImage
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable --now quant-infra-net-pro
~/bin/quantctl status          # 自检；先看第一个 [XX] 项和它的修复提示
```

服务器是 Windows 的话：安装 `QuantInfraNetPro.Web-win-x64-Setup.exe`，之后从开始菜单启动。Windows 上的控制命令这样运行：
`"%LOCALAPPDATA%\QuantInfraNetPro.Web\current\Quant.Infra.Net.Pro.Web.exe" ctl <命令>`。

### 第 2 步，办法 A：SSH 隧道（最简单）

在**你的电脑**上打开终端（Windows 10/11 用 PowerShell，自带 `ssh`），运行：

```bash
ssh -N -L 8443:127.0.0.1:8443 <USER>@<SERVER-IP>
```

- 窗口看起来像卡住了，这是正常的：用 AWLPilot 期间**不要关这个窗口**。
- 在你电脑的浏览器打开 **`https://127.0.0.1:8443`**。
- 如果你这台电脑自己也在 8443 跑着 AWLPilot，就换一个本机端口，比如 `-L 9443:127.0.0.1:8443`，然后打开 `https://127.0.0.1:9443`。**这时嘉信登录需要第 3 步里多出来的粘贴操作。**
- 按 Ctrl+C 停止。SSH 尽量用密钥登录（`ssh-keygen`，再 `ssh-copy-id <USER>@<SERVER-IP>`）。

### 第 2 步，办法 B：远程面板（直接打开服务器地址）

在**服务器**上：

```bash
~/bin/quantctl bind --list                    # 列出服务器的内网地址
~/bin/quantctl remote on --bind <SERVER-IP>   # 只选一个内网地址；会打印一次性口令
sudo systemctl restart quant-infra-net-pro
~/bin/quantctl info                           # 打印完整地址（含随机入口路径）
~/bin/quantctl doctor                         # 自检和防火墙建议
```

- 服务器有多个内网地址时（比如局域网加虚拟网），程序不会猜，请用 `--bind` 指定你的电脑能到达的那个。
- 服务器开着防火墙的话，8443 **只对你自己的网络放行**。Ubuntu 例如：`sudo ufw allow from <YOUR-NETWORK-CIDR> to any port 8443 proto tcp`。
- 随时可用 `~/bin/quantctl passwd` 改口令。
- 在你的电脑上打开 `quantctl info` 给的地址：`https://<SERVER-IP>:8443/<SECRET-PATH>`。路径错了会故意显示 404。随机路径和口令都要像密码一样保管。

### 浏览器的证书警告

程序用自己生成的本地证书，所以浏览器第一次会提示"不安全"。对你自己的电脑，这是预期的：点继续即可；也可以一次性安装它的证书：在服务器上运行 `~/bin/quantctl trust-ca`，按提示在你的电脑上操作。

### 第 3 步：在浏览器里完成首次设置（一次）

1. **接受 EULA**（必须你自己点）。
2. 在 `/start` 或 `/settings` **开始试用或输入 License**。试用需要邮箱验证。
3. 在 `/settings` **填入嘉信 App Key、App Secret 和账号**。先要有一个已批准的嘉信个人开发者应用：[开发者注册指南](Charles%20Schwab%20Developer%20Registration%20Guide.md)。
4. **登录嘉信**：打开 `/signin`，用你自己的嘉信账号登录。
5. 登录嘉信后，浏览器会跳到一个**打不开**的页面，地址类似 `https://127.0.0.1:8443/?code=...`。
   - **办法 A 且用 8443：** 它会通过隧道到达服务器，自动完成。
   - **办法 B，或用了其他本机端口：** 这是正常的。从地址栏复制**整条地址**，在面板里打开 `/lan/finish-auth`，粘贴提交。授权码几分钟内失效。
6. 打开 `/Rebalance`，设目标权重，先检查动作单再决定执行。

### 日常使用与维护

| 事项 | 怎么做 |
|---|---|
| 使用 | 办法 A：开隧道，打开 `https://127.0.0.1:8443`。办法 B：打开 `quantctl info` 给的地址。 |
| 嘉信登录每 7 天过期 | 面板会出现横幅，点 **Sign In**，重复第 4、5 步。 |
| 升级 | 美股休市时打开 `/update`，点 **Upgrade & Restart**。 |
| 健康检查 | 服务器上运行 `~/bin/quantctl status` 和 `~/bin/quantctl doctor` |
| 日志 | `~/bin/quantctl log --tail 30` |
| 清理磁盘 | 升级后，旧版 AppImage 会以 `*.old` 留在新版旁边以便回滚。不需要时可删掉旧的。 |

### 故障排查

| 现象 | 处理 |
|---|---|
| 打开 `127.0.0.1` 提示"无法访问此网站 / 拒绝连接" | 办法 A：隧道窗口被关了，重新开。再检查服务器上程序是否在运行：`systemctl status quant-infra-net-pro`。 |
| 打不开 `<SERVER-IP>:8443` | 远程面板没开（看 `quantctl info`）、程序没重启、防火墙拦了 8443，或你们不在同一个网络。 |
| 页面显示 404 | 随机路径不对，重新运行 `quantctl info`。 |
| 面板提示"Authorization has expired" | 首次使用或 7 天后都是正常的，重新登录嘉信（第 4、5 步）。 |
| `ssh: Permission denied` | 用户名不对，或服务器没有给你开放密钥或密码登录。 |
