# 2026-04-16｜阶段三：双人隔离、多代理收敛、Cron 与媒体链路验收

这篇记录 phase-2 之后的继续落地：把单人可用升级为“双人可隔离 + 系统任务可执行 + 媒体可送达”。

目标：
- 我与老婆的 agent/workspace 彻底隔离
- main 路由稳定，不因子会话限制卡死
- OpenClaw 内置 cron 真正可创建、可查看、可运行
- Telegram 图片/文件发送链路可验收（不是只返回路径）

---

## 1) 双人隔离：wife 专属 agent 组已建成

新增并校验：
- `main_wife`
- `researcher_wife`
- `coder_wife`
- `writer_wife`
- `polisher_wife`

工作区采用独立目录：
- `/root/.openclaw/wife/main`
- `/root/.openclaw/wife/researcher`
- `/root/.openclaw/wife/coder`
- `/root/.openclaw/wife/writer`
- `/root/.openclaw/wife/polisher`

结论：
- agent 隔离结构已成立
- 后续只需把老婆渠道（微信）绑定到 `main_wife`，即可完成记忆/任务面隔离

---

## 2) 分派策略收敛：系统任务回归 main 直执行

实践中发现：
- Telegram 渠道下，`main -> sessions_spawn(ops)` 会遇到可见性限制
- 典型报错：`Session not visible from this sandboxed agent session.`

最终策略调整：
- `cron/config/gateway/docker/文件迁移` 这类系统任务由 `main` 直接执行
- `researcher/coder/writer/polisher` 继续承担各自专业任务
- `ops` 作为可选运维专员保留，不再成为系统任务唯一入口

效果：
- 避免“强制经 ops”导致任务卡死
- 保持自然语言下达后可快速落地

---

## 3) Cron 验收：报餐双提醒已落地

已存在任务（验证通过）：
- `meal-reminder-1000`（`0 10 * * *`）
- `meal-reminder-1150`（`50 11 * * *`）

`openclaw cron list` 输出可见、状态正常（含 `ok/idle` 与 next run）。

已完成闭环：
- 创建 -> 列表可见 -> 运行可查 -> 结果可回传

---

## 4) 媒体链路验收：Telegram 可真实收图

关键验收命令（已成功）：

```bash
openclaw message send --channel telegram --target 2047210273 \
  --media /root/.openclaw/workspace/_deliverables/current/calendar_2026_05.png \
  --message "media-smoke" --json
```

成功回执（摘要）：
- `ok: true`
- `messageId: 14975`

结论：
- 平台“发媒体”能力正常
- 之前“只说已生成但未发图”属于执行流程问题，不是通道能力问题

---

## 5) 存储策略 v3：交付与中间文件分离

本人目录：
- 最终交付：`/root/.openclaw/workspace/_deliverables/current`
- 中间产物：`/root/.openclaw/workspace/tmp`、`/root/.openclaw/workspace/runs`

老婆目录：
- 最终交付：`/root/.openclaw/wife/main/_deliverables/current`
- 中间产物：`/root/.openclaw/wife/main/tmp`、`/root/.openclaw/wife/main/runs`

执行约束：
- 最终文件禁止落 `/root` 与 `/tmp`
- 迁移时输出“旧路径 -> 新路径”清单

---

## 6) 阶段结论

截至 2026-04-16，环境已从“单人可用”升级到“家庭双人可扩展”：
- 架构：双组 agent 隔离完成
- 调度：系统任务不再卡在 spawn 链路
- 自动化：cron 任务可持续运行
- 交付：媒体链路已通过真实发送验证

这套状态可以进入下一阶段：微信绑定与老婆侧路由接入。

---

记录时间：2026-04-16  
阶段标签：phase-3 routing + cron + media
