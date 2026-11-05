# 知识库（knowledge/）

存放可复用的业务知识、产品事实与操作规范，供 OpenClaw / 客服营销助手检索与引用。

## 用途
- 沉淀稳定事实，避免只靠会话记忆
- 给人设、话术、KPI 提供统一事实来源
- 修正结论优先写回这里，再同步到 `memory/corrections.md`

## 目录规范
| 文件/类型 | 说明 |
|-----------|------|
| `README.md` | 本目录说明 |
| `product.md` | 产品资料 |
| `scripts.md` | 话术与常见回复 |
| `*.md` | 其他专题知识（按主题命名，小写短横线） |

## 撰写约定
1. 标题清晰；先结论，后细节
2. 用列表/表格承载可核对信息
3. **禁止**写入 API Key、session、密码、PAT
4. 过时内容标 `status: deprecated`，不要直接删历史关键事实

## 示例条目骨架
```markdown
# 主题名
- status: active
- owner: 运营/客服
- updated: YYYY-MM-DD

## 要点
- …

## 细节
…
```
