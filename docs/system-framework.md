# 系统框架备份（System Framework）

> 用途：换机器 / 重装云电脑时快速理解与恢复整套「龙虾 / OpenClaw + 知识库 + Telegram Userbot」体系。  
> 更新：2026-11-05（Asia/Hong_Kong）  
> 仓库：`https://github.com/duongvanbon04121989-pixel/ai-system-knowledge`  
> **禁止**把 API Key、PAT、Telegram `api_hash`、session、Gateway token 写入本仓库。

---

## 1. 整体架构

```text
┌─────────────────────────────────────────────────────────┐
│ 大脑层  Models / Personas / Memory                       │
│  Uncensored DeepSeek Flash（主） / Pro（辅）              │
│  人设、纠正记录、长期记忆（助手 memory + 知识库 memory/）   │
└──────────────────────────▲──────────────────────────────┘
                           │ 决策 / 提示
┌──────────────────────────┴──────────────────────────────┐
│ 调度层  龙虾调度助手 + Routines + OpenClaw Gateway        │
│  Grok Bot 例行任务（Git 同步、GDrive 备份）               │
│  OpenClaw Gateway :18789（local）                        │
└──────────────────────────▲──────────────────────────────┘
                           │ 下发任务
┌──────────────────────────┴──────────────────────────────┐
│ 执行层  OpenClaw / tg-manager / Shell 脚本               │
│  openclaw CLI、gateway start/stop                        │
│  /workspace/tg-manager（Telethon Userbot，最多约 5 账号） │
└──────────────────────────▲──────────────────────────────┘
                           │ 读写资产
┌──────────────────────────┴──────────────────────────────┐
│ 资产层  知识与运维数据                                    │
│  /workspace/knowledge → GitHub + rclone→Google Drive     │
│  /workspace/openclaw-ops（KPI / personas / logs）         │
└──────────────────────────▲──────────────────────────────┘
                           │ 跑在
┌──────────────────────────┴──────────────────────────────┐
│ 环境层  云电脑 Box                                        │
│  Debian、Node ~/.local/node、OpenClaw ~/.local、8G swap   │
│  密钥目录 ~/.config/*（600）、可选 SOCKS5/HTTP 代理        │
└─────────────────────────────────────────────────────────┘
```

| 层级 | 职责 | 关键组件 |
|------|------|----------|
| 大脑层 | 模型选择、人设、记忆与纠正 | DeepSeek Flash/Pro（待接入）、`prompts/`、`memory/`、助手 durable memory |
| 调度层 | 定时与会话调度、Gateway | 龙虾调度助手、Routines、`openclaw gateway` |
| 执行层 | 实际操作 Telegram / 脚本 / Agent | `tg-manager`、OpenClaw agents、ops 脚本 |
| 资产层 | 可版本化知识与备份 | `knowledge` Git 仓、GDrive、`openclaw-ops` |
| 环境层 | 算力、运行时、密钥与网络 | Box（约 8 核 / 15G RAM / 128G 盘）、Node 26、swap、proxy |

**产品偏好（业务约束）**

- Telegram：优先 **Userbot（MTProto / Telethon）**，不用官方 Bot Token 做主控。
- 模型：Uncensored DeepSeek **Flash 为主、Pro 为辅**（多经 Venice 一类 API，密钥另存）。
- 并发：目标平滑支撑约 **5** 个 Telegram 账号。

---

## 2. 已完成清单

| 项 | 状态 | 说明 |
|----|------|------|
| 知识库 Git 仓库 | ✅ | `/workspace/knowledge` → `origin` GitHub `ai-system-knowledge` |
| 知识库模板 | ✅ | `knowledge/`、`prompts/`、`memory/`、`config/` 起步模板（如 `fd3f4cb`） |
| Git 自动同步 | ✅ | Routine「Knowledge Git 自动同步」+ `/home/box/.config/knowledge-git/sync.sh`（约 09–21 点每小时） |
| Google Drive 备份 | ✅ 已配置 / 例行待跑 | rclone remote `gdrive:`，脚本 `gdrive-backup.sh` → `gdrive:ai-system-backup/knowledge`；Routine 每日约 10:00 |
| OpenClaw 安装 | ✅ | OpenClaw **2026.11.1**，Node **v26.12.1**（`~/.local/node`） |
| Gateway | ✅ 运行中 | `127.0.0.1:18789`，`openclaw-gateway-start` / `stop` |
| 运维骨架 | ✅ | `/workspace/openclaw-ops`（KPI、personas、accounts、logs） |
| 8G swap | ✅ | 缓解约 15G 内存压力 |
| tg-manager | ✅ 代码就绪 | `/workspace/tg-manager`；账号 `acct1` 已添加 |
| Telegram 凭据落盘 | ✅ | `~/.config/tg-manager/env`（mode **600**，仅环境变量名进逻辑） |
| Telegram 发码登录 | ❌ 阻塞 | 见第 4 节 |
| DeepSeek / Venice API | ❌ 未接入 | 见第 4 节 |
| ClawHub telegram-userbot 插件 | ❌ 不可用 | 与 OpenClaw 2026.11.1 SDK/manifest 不兼容；改走 tg-manager |

---

## 3. 关键路径与配置位置

### 3.1 代码与数据

| 路径 | 用途 |
|------|------|
| `/workspace/knowledge` | 本知识库 Git 工作树 |
| `/workspace/openclaw-ops` | KPI、人设、账号登记、日志 |
| `/workspace/tg-manager` | Telethon 多账号 CLI / DB / sessions |
| `/workspace/README.md` | workspace 总览（若有） |
| `~/.local/node` | Node 运行时 |
| `~/.local/bin/openclaw` | OpenClaw CLI |
| `~/.local/bin/openclaw-gateway-start` | 后台启动 Gateway |
| `~/.local/bin/openclaw-gateway-stop` | 停止 Gateway |
| `~/.openclaw/openclaw.json` | Gateway / plugins / agents 配置（含敏感 token，勿提交） |
| `~/.openclaw/gateway.pid` | Gateway PID 文件 |
| `/workspace/openclaw-ops/logs/openclaw-gateway.log` | Gateway 日志 |

### 3.2 密钥与凭据（永不进 Git）

| 路径 | 内容 | 权限约定 |
|------|------|----------|
| `/home/box/.config/tg-manager/env` | `TG_API_ID` / `TG_API_HASH` | `600` |
| `/home/box/.config/knowledge-git/github_pat` | GitHub PAT（供 credential helper） | `600` |
| `/home/box/.config/knowledge-git/git-credential-knowledge.sh` | Git 凭据助手 | `700` |
| `/home/box/.config/knowledge-git/git-askpass.sh` | askpass | `700` |
| `/home/box/.config/rclone/rclone.conf` | Google Drive OAuth remote `gdrive` | `600` |
| `~/.openclaw/openclaw.json` | Gateway auth token 等 | 仅本机 |
| `/workspace/tg-manager/sessions/*.session` | Telethon session | `600` / 目录 `700` |

### 3.3 自动化脚本与例程

| 脚本 / 例程 | 作用 |
|-------------|------|
| `/home/box/.config/knowledge-git/sync.sh` | `git pull --rebase` → 有变更则 commit → `push`；日志：`openclaw-ops/logs/knowledge-git-sync.log` |
| `/home/box/.config/knowledge-git/gdrive-backup.sh` | `rclone sync` 知识库到 Drive；日志：`openclaw-ops/logs/gdrive-backup.log` |
| Routine: Knowledge Git 自动同步 | Cron（HK）：`0 9-21 * * *`，可 ±15 分钟 |
| Routine: Knowledge Google Drive 备份 | Cron（HK）：每日约 `0 10 * * *` |

### 3.4 Telegram（tg-manager）

```bash
cd /workspace/tg-manager
set -a; source /home/box/.config/tg-manager/env; set +a
.venv/bin/python3 cli.py account list
.venv/bin/python3 cli.py account add <label> --phone +<E.164> [--proxy socks5://host:port]
.venv/bin/python3 cli.py account login <label> --send-code
.venv/bin/python3 cli.py account login <label> --code <OTP> [--password-env TG_2FA_<LABEL>]
```

当前：`acct1`，手机号脱敏 `+66*******72`，`status=pending_login`，`creds_present=True`。

---

## 4. 当前阻塞点

### 4.1 Telegram MTProto（发验证码失败）

- **现象**：TCP 能连 Telegram DC（如 `149.154.167.x:443`），但 Telethon 生成 auth key 时 `IncompleteReadError`（读到垃圾包长度），直连与经本机 `127.0.0.1:8791` HTTP CONNECT 均失败。
- **含义**：云电脑出口对 MTProto 握手有干扰 / 污染，不是 `api_id` 填错。
- **解法**：为 `acct1` 配置**可信** `socks5://` 或 HTTP CONNECT 代理后再 `--send-code`；不要用不明公共代理传登录流量。
- **下一步**：代理就绪 → 发码 → 用户回填验证码 →（如有）2FA → `account` 状态检查。

### 4.2 大脑层 API（Venice / Uncensored DeepSeek）

- Flash / Pro 的 **base URL + API Key 尚未写入**安全配置；OpenClaw Gateway 已起，但模型侧未完成 onboard。
- **解法**：用户提供后写入仅本机可读配置（勿进本 Git），再在 OpenClaw / 调度助手侧绑定主辅模型。

### 4.3 OpenClaw 原生 Userbot 插件

- ClawHub `telegram-userbot` 与 **2026.11.1** 不兼容 → 已弃用该路径；以 **tg-manager** 为 Userbot 执行层，待插件生态跟上再评估桥接。

---

## 5. 重建步骤概要（换机器）

1. **准备环境**  
   - Debian 类 Box；内存紧张时加 **8G swap**。  
   - 安装 Node（建议对齐 `~/.local/node` 的 v26.x 手工 tarball 方式，避免坏掉的系统 apt Node）。

2. **恢复秘密（先于业务）**  
   - 重建 `~/.config/tg-manager/env`、`knowledge-git/github_pat`、`rclone.conf`、DeepSeek/Venice key、Gateway token。  
   - 全部 `chmod 600`；确认不在任何 Git 跟踪路径下。

3. **拉起资产层**  
   ```bash
   git clone https://github.com/duongvanbon04121989-pixel/ai-system-knowledge.git /workspace/knowledge
   # 配置 credential helper 指向 knowledge-git 脚本后：
   /home/box/.config/knowledge-git/sync.sh   # 验证 push 通路
   rclone listremotes   # 应有 gdrive:
   /home/box/.config/knowledge-git/gdrive-backup.sh
   ```

4. **安装 OpenClaw**  
   - 恢复 `~/.local` 中的 CLI / 或按官方方式重装同大版本。  
   - 恢复 `~/.openclaw/openclaw.json`（或向导重建后改 `gateway.mode=local`）。  
   - `openclaw-gateway-start`，确认 `127.0.0.1:18789` 监听。

5. **运维目录**  
   - 恢复或重建 `/workspace/openclaw-ops`（可用本仓库文档 + 空 KPI/CSV 骨架）。

6. **Telegram Userbot**  
   - 恢复 `/workspace/tg-manager`（含 `.venv` 或 `pip install` 依赖）。  
   - `source` env → `account add` → **先设 proxy** → `--send-code` → `--code`。  
   - 旧 `sessions/*.session` 若仍有效可跳过发码（先 `connect` 测 `is_user_authorized`）。

7. **调度**  
   - 在助手侧重建两条 Routine（Git 小时同步、GDrive 日备），指向上述脚本。  
   - 确认 龙虾调度助手 人设 / 记忆中的业务偏好仍有效。

8. **验收**  
   - `git -C /workspace/knowledge status` 干净且能 push。  
   - `rclone lsf gdrive:ai-system-backup/knowledge` 有内容。  
   - Gateway 健康；`tg-manager account list` 中账号 `active`。  
   - 模型探测：Flash/Pro 各一次最小补全。

---

## 6. 目录结构说明

### 6.1 知识库仓库（本仓）

```text
/workspace/knowledge/
├── README.md                 # 仓说明与约定
├── docs/
│   └── system-framework.md   # 本文件：系统框架备份
├── knowledge/                # 业务知识条目
│   ├── README.md
│   ├── product.md
│   └── scripts.md
├── prompts/
│   └── system.md             # 系统提示 / 性格相关
├── memory/
│   └── corrections.md        # 纠正与学习记录
└── config/
    └── settings.md           # 非密钥配置说明
```

### 6.2 工作区与运维

```text
/workspace/
├── knowledge/          # Git 知识库（上表）
├── openclaw-ops/
│   ├── README.md
│   ├── accounts/registry.csv
│   ├── kpi/daily.csv, schema.md
│   ├── personas/default.md
│   └── logs/           # gateway / git-sync / gdrive-backup
└── tg-manager/         # Userbot 工具（通常不进 knowledge 仓）
    ├── cli.py, tgm/, schema.sql, tg.db
    ├── sessions/       # 敏感
    └── .venv/
```

### 6.3 本机配置树（摘录）

```text
/home/box/.config/
├── knowledge-git/      # sync.sh, gdrive-backup.sh, PAT, git helpers
├── rclone/rclone.conf
└── tg-manager/env

/home/box/.openclaw/    # openclaw.json, gateway.pid, agents, extensions
/home/box/.local/
├── node/               # Node v26.x
└── bin/                # openclaw, gateway-start/stop
```

---

## 附录 A — 快速健康检查

```bash
# 资源
free -h; swapon --show

# Gateway
ss -ltnp | grep 18789 || true
tail -n 30 /workspace/openclaw-ops/logs/openclaw-gateway.log

# 知识库
git -C /workspace/knowledge status -sb
git -C /workspace/knowledge remote -v

# Drive
rclone listremotes | grep gdrive

# Telegram（需已 source env；未配代理时 connect 可能失败）
cd /workspace/tg-manager && .venv/bin/python3 cli.py account list
```

## 附录 B — 安全红线

1. 秘密只活在 `~/.config/**` 或 session 文件，**绝不** commit。  
2. `sync.sh` 已尝试排除 `*.env` / `*token*` / `*secret*`；仍勿把密钥放进 `/workspace/knowledge`。  
3. Telegram 登录流量只走**自己的**代理。  
4. 分享本文件或导出备份前，再扫一遍是否含 token 形态字符串。

---

*Maintainer: 龙虾调度助手 · 与 OpenClaw / tg-manager 部署状态同步维护*
