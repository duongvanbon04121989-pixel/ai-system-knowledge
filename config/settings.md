# 配置说明模板

- status: draft
- updated: YYYY-MM-DD

> 本文件只描述**配置项含义与推荐值**，不存放真实密钥。密钥放本机受控文件或环境变量。

## OpenClaw
| 项 | 推荐 | 说明 |
|----|------|------|
| `gateway.mode` | `local` | 本机网关 |
| `agents.defaults.maxConcurrent` | `3`（先稳） | 扩到 5 账号前勿盲目拉高 |
| 主力模型 | Uncensored DeepSeek Flash | 日常客服/营销 |
| 补充模型 | DeepSeek Pro | 复杂推理/长文 |
| 记忆检索 | `none` 或自建 | 无嵌入密钥时用关键词/文件 |

## Telegram Userbot（真实账号）
| 项 | 说明 |
|----|------|
| `api_id` / `api_hash` | 来自 my.telegram.org；只放环境变量 |
| session | 本机加密保存，禁止提交 Git |
| 并发策略 | 先 1 个号跑通 → 2 → 5；账号内串行 |
| 风险 | 可能触 ToS；只用专用号 |

## 知识库同步
| 项 | 值 |
|----|----|
| 本地路径 | `/workspace/knowledge` |
| 远程 | `github.com/duongvanbon04121989-pixel/ai-system-knowledge` |
| 定时 | 香港时区 9–21 点每小时（例程） |

## KPI（占位）
- 日报字段见运维目录 `kpi/daily.csv`（若使用）
- 关注：在线账号数、收发消息、Flash/Pro 调用、FloodWait、错误数

## 变更记录
| 日期 | 变更 | 操作者 |
|------|------|--------|
| 2026-11-05 | 初始化模板 | 龙虾调度助手 |
