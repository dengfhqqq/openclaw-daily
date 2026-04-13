# 2026-04-13｜官方版 OpenClaw 极简清单（20 条命令）

只做命令，不看解释，适合重装或迁移（例如 Mac mini M4）。

```bash
# 01
npm -g rm openclaw-cn clawdbot openclaw 2>/dev/null || true
# 02
pnpm -g remove openclaw-cn clawdbot openclaw 2>/dev/null || true
# 03
npm -g install openclaw@latest
# 04
openclaw --version
# 05
openclaw onboard

# 06
openclaw status
# 07
openclaw gateway probe
# 08
openclaw auth login --provider openai-codex
# 09
openclaw models set-default openai-codex/gpt-5.4
# 10
openclaw gateway restart

# 11
openclaw configure --section web
# 12
openclaw config get tools.web --json

# 13
openclaw agents add researcher --workspace ~/.openclaw/workspace-researcher --non-interactive
# 14
openclaw agents add coder --workspace ~/.openclaw/workspace-coder --non-interactive
# 15
openclaw agents add writer --workspace ~/.openclaw/workspace-writer --non-interactive
# 16
openclaw agents add polisher --workspace ~/.openclaw/workspace-polisher --non-interactive

# 17
openclaw agents list --json
# 18
openclaw models status
# 19
openclaw agent --agent main --session-id smoke-$(date +%s) --thinking off --message "只回复 OK"
# 20
openclaw agent --agent main --session-id webok-$(date +%s) --thinking off --message "默认广东话。联网检索今日Apple办公相关热点，返回一句结论+两条来源(日期+链接)+若非当天则写近期。"
```

## TG 验收口令（可直接发）

```text
默认用广东话。任务：搜今日一个 Apple 办公相关热点，写 120 字快讯并润色。你要自动分派并执行，先输出一行 [delegation]，再输出最终稿；必须附两条来源（日期+链接）；若非当天发布请明确写“近期”。
```

## 合格标准
- 有 `[delegation]`
- 有最终稿
- 有来源日期+链接
- 能标注“今日/近期”口径

