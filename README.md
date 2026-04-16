# openclaw-daily

这份仓库记录我在 NAS / Linux 上部署官方 OpenClaw（非 -cn）并用于 AI 办公的实战过程。

## 目录
- [2026-04-13｜官方版 OpenClaw：onboard 后到稳定可用（傻瓜式命令）](./diary/2026-04-13-openclaw-official-onboard-to-stable.md)
- [2026-04-13｜官方版 OpenClaw 极简清单（20 条命令）](./diary/2026-04-13-openclaw-official-quick-checklist.md)
- [2026-04-13｜阶段二：多 Agent 技能分工 + 最小权限收敛（已验收）](./diary/2026-04-13-phase2-skills-and-least-privilege.md)
- [2026-04-16｜阶段三：双人隔离、多代理收敛、Cron 与媒体链路验收](./diary/2026-04-16-phase3-routing-cron-media.md)

## 当前稳定基线（2026-04-16）
- OpenClaw: `2026.4.11`
- Gateway: `local + loopback`
- Model: `openai-codex/gpt-5.4`（OAuth 登录）
- Agents: `main, researcher, coder, writer, polisher, ops` + `*_wife` 隔离组
- Web: `tools.web.search=tavily + openaiCodex(cached) + web_fetch`
- Channel: Telegram 已接通
- Cron: 报餐提醒双任务（`10:00` + `11:50`）已落地
- Media: CLI 附件发送链路已验证（返回 `messageId`）

## 快速复现（超简版）
```bash
# 1) 安装官方版
npm -g rm openclaw-cn clawdbot openclaw 2>/dev/null || true
npm -g install openclaw@latest
openclaw --version

# 2) 走 onboard（交互式）
openclaw onboard

# 3) 核验
openclaw status
openclaw gateway probe
openclaw models status
openclaw agents list --json
```

## 说明
- 本仓库以“可复制、可回滚、可验证”为原则，只记录我实际跑通过的步骤。
- 详细步骤、排错与命令见日记文件。

