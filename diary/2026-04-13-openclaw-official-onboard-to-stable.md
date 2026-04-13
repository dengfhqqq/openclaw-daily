# 2026-04-13｜官方版 OpenClaw：onboard 后到稳定可用（傻瓜式命令）

适用场景：你已经 `openclaw onboard` 走完，但要把环境收敛成“可稳定在 Telegram 自然语言发号施令”的状态。

目标状态：
- 主路由 agent（main）+ 多 specialist（researcher/coder/writer/polisher）
- 默认广东话回复
- 可联网检索并带来源
- 不再出现只 `completed` 不出正文（通过约束降低概率）

---

## A. 一次性前置（官方版、非 -cn）

```bash
# 清理旧包（可重复执行）
npm -g rm openclaw-cn clawdbot openclaw 2>/dev/null || true
pnpm -g remove openclaw-cn clawdbot openclaw 2>/dev/null || true

# 安装官方版
npm -g install openclaw@latest

# 验证
which openclaw
openclaw --version
openclaw doctor
```

期望：版本类似 `OpenClaw 2026.4.11`。

---

## B. onboard 后必做核验（3 分钟）

```bash
openclaw status
openclaw gateway probe
openclaw config get gateway.mode
openclaw config get tools.profile
openclaw models status
openclaw agents list --json
```

我这次稳定值参考：
- `gateway.mode=local`
- `gateway.bind=loopback`
- `tools.profile=coding`
- 默认模型 `openai-codex/gpt-5.4`

---

## C. 网关登录/连接失败（最常见）

### C1. 浏览器报 secure context / device identity

在本地电脑开 SSH 隧道（不要关）：
```bash
ssh -N -L 18789:127.0.0.1:18789 root@<NAS_IP>
```

浏览器只用：
- `http://127.0.0.1:18789/` 或 `http://localhost:18789/`
- 不要直接用远端 IP 网页地址

### C2. CLI 报 `pairing required`

```bash
openclaw devices list
openclaw devices approve <request-id>
openclaw devices list
```

---

## D. 模型切到官方 OAuth（避免第三方中转不稳）

> 你若继续用第三方 API 可跳过，但稳定性通常不如官方 OAuth。

```bash
# 按向导登录 openai-codex
openclaw auth login --provider openai-codex

# 验证模型
openclaw models list --provider openai-codex
openclaw models status

# 建议设默认（如果向导未自动设）
openclaw models set-default openai-codex/gpt-5.4
openclaw gateway restart
```

验证烟测：
```bash
openclaw agent --agent main --session-id smoke-$(date +%s) --thinking off --message "只回复 OK"
```

---

## E. 启用联网检索（Tavily + Codex native search）

```bash
openclaw configure --section web
```

推荐选择：
- Enable `web_search`: Yes
- Enable native Codex web search: Yes
- Mode: `cached`
- Managed provider: `Tavily`
- Enable `web_fetch`: Yes

核验：
```bash
openclaw config get tools.web --json
```

期望示例：
```json
{
  "search": {
    "enabled": true,
    "provider": "tavily",
    "openaiCodex": { "enabled": true, "mode": "cached" }
  },
  "fetch": { "enabled": true }
}
```

---

## F. 创建多 agent（CLI 正式命令）

```bash
openclaw agents add researcher --workspace ~/.openclaw/workspace-researcher --non-interactive
openclaw agents add coder      --workspace ~/.openclaw/workspace-coder      --non-interactive
openclaw agents add writer     --workspace ~/.openclaw/workspace-writer     --non-interactive
openclaw agents add polisher   --workspace ~/.openclaw/workspace-polisher   --non-interactive

openclaw agents list --json
```

---

## G. 给 main 加“强输出约束”（防止只 completed）

编辑：`~/.openclaw/workspace/AGENTS.md`

补充以下段落（可直接粘贴）：

```md
## Hard Output Contract
- Every run must return visible final text.
- Never end a run with empty output.
- 必须在当前会话输出最终可见文本；禁止只结束 run 或只发 announce。
- If any subtask/tool fails, still return:
  1) failure reason
  2) what succeeded
  3) next actionable step
- Minimum final reply length: 60 Chinese characters unless user explicitly asks for a short answer.

## Delegation Reporting Contract
- 当任务涉及 researcher / writer / polisher 时，先输出一行分派清单：
  [delegation] researcher=..., writer=..., polisher=...
- 最后必须输出“最终稿”段落，禁止只输出中间日志或空结果。

## Source Guardrail
- 涉及“今日/最新”资讯，必须给出至少 2 条可访问来源链接与日期。
- 若非当天发布，必须明确标注“近期”。
- 若未找到可靠来源，必须明确写“未验证”，禁止当成事实陈述。
```

保存后无需重启，建议新会话验证：
```bash
openclaw agent --agent main --session-id test-$(date +%s) --thinking off --message "默认用广东话。先给[delegation]，再给最终稿。"
```

---

## H. Telegram 实战模板（自然语言，不点名也能分派）

发给 main：

```text
默认用广东话。任务：搜今日一个 Apple 办公相关热点，写 120 字快讯并润色。你要自动分派并执行，先输出一行 [delegation]，再输出最终稿；必须附两条来源（日期+链接）；若非当天发布请明确写“近期”。
```

合格标准：
- 有 `[delegation]`
- 有“最终稿”
- 有来源（日期+链接）
- 对“今日/近期”口径有明确说明

---

## I. 常见故障速查

### I1. `Context overflow`
- 常见于第三方小上下文模型（如 16k）
- 解决：改用 `openai-codex/gpt-5.4`（大上下文）+ 新会话 `/new`

### I2. `GatewayClientRequestError: pairing required`
- 运行 `openclaw devices list` + `openclaw devices approve <id>`

### I3. `control ui requires device identity (use HTTPS or localhost secure context)`
- 用 SSH 隧道 + 本地 `127.0.0.1:18789` 打开控制台

### I4. 工具名不一致（例如说没有 `web_search_query`）
- 不要强绑函数名；用“结果验证法”：要求返回可核验链接与日期

---

## J. Mac mini M4 迁移照抄版（后续办公机）

```bash
# 1) 安装
npm -g install openclaw@latest
openclaw --version

# 2) 向导
openclaw onboard

# 3) 登录官方模型
openclaw auth login --provider openai-codex
openclaw models set-default openai-codex/gpt-5.4

# 4) 开 web
openclaw configure --section web

# 5) 建团队
openclaw agents add researcher --workspace ~/.openclaw/workspace-researcher --non-interactive
openclaw agents add coder      --workspace ~/.openclaw/workspace-coder      --non-interactive
openclaw agents add writer     --workspace ~/.openclaw/workspace-writer     --non-interactive
openclaw agents add polisher   --workspace ~/.openclaw/workspace-polisher   --non-interactive

# 6) 验收
openclaw status
openclaw models status
openclaw agents list --json
openclaw config get tools.web --json
```

---

## K. 提交到 GitHub

```bash
cd ~/openclaw-daily
git add README.md diary/2026-04-13-openclaw-official-onboard-to-stable.md
git commit -m "docs: add official OpenClaw onboard-to-stable foolproof guide (2026-04-13)"
git push
```

---

记录时间：2026-04-13
环境：NAS Linux + Telegram + OpenClaw official 2026.4.11
