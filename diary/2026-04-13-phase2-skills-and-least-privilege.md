# 2026-04-13｜阶段二：多 Agent 技能分工 + 最小权限收敛（已验收）

这篇是上一阶段（完成 onboard 到可用）之后的“生产化加固”记录。

目标：
- 不让所有 agent 全权限乱跑
- 各司其职（skills 分工）
- 保持自然语言指挥可用
- 通过实测验收

---

## 1) 权限收敛（全局）

执行：

```bash
openclaw config set tools.profile minimal
openclaw config set agents.defaults.sandbox.mode all
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.ask on-miss
```

说明：
- `tools.exec.askFallback` 在 2026.4.11 报 `Unrecognized key`，本版本无此字段，已跳过。

主路由自检通过：

```bash
openclaw agent --agent main --thinking off --message "请只说明你会做任务分派与汇总，不执行本地命令。"
```

返回（摘要）：
- “只负责任务分派/汇总，不执行本地命令。”

---

## 2) skills 准备与安装

先做检查：

```bash
openclaw skills --help
openclaw skills list
openclaw skills check
```

本次重点 skills：
- researcher：`tavily`, `session-logs`, `nano-pdf`
- coder：`github`, `gh-issues`, `mcporter`, `session-logs`
- writer：`nano-pdf`, `session-logs`
- polisher：`session-logs`
- main：`taskflow`, `healthcheck`, `node-connect`

安装与补依赖结果：
- `github` / `gh-issues` / `mcporter` / `nano-pdf` / `session-logs` 已安装并可用
- `session-logs` 缺 `rg`，补装后转 Ready
- `gh-issues` 缺 `gh`，补装后转 Ready
- `summarize` 为 bundled，`skills install summarize` 404 属预期，不影响当前流程

---

## 3) 按 Agent 指定 skills（核心）

先清空默认：

```bash
openclaw config set agents.defaults.skills '[]' --strict-json
```

再按索引写入：

```bash
openclaw config set agents.list[0].skills '["taskflow","healthcheck","node-connect"]' --strict-json
openclaw config set agents.list[1].skills '["tavily","session-logs","nano-pdf"]' --strict-json
openclaw config set agents.list[2].skills '["github","gh-issues","mcporter","session-logs"]' --strict-json
openclaw config set agents.list[3].skills '["nano-pdf","session-logs"]' --strict-json
openclaw config set agents.list[4].skills '["session-logs"]' --strict-json
openclaw gateway restart
```

核验：

```bash
openclaw config get agents.defaults.skills --json
openclaw config get agents.list --json
```

结果：
- `agents.defaults.skills = []`
- 每个 agent 都有独立 skills 列表（已写入成功）

---

## 4) 验收测试（通过）

### main（路由）

```bash
openclaw agent --agent main --thinking off --message "请用 taskflow 方式给出一个三步任务流（只文字，不执行命令）。"
```

通过：输出了 3 步任务流。

### researcher（检索）

```bash
openclaw agent --agent researcher --thinking off --message "请联网检索今日一个AI办公热点，附2条来源日期+链接。"
```

通过：返回热点 + 2 条来源（日期+链接）。

### coder（GitHub能力）

```bash
openclaw agent --agent coder --thinking off --message "请说明你可用的 GitHub 相关能力（github, gh-issues, mcporter）并给一条示例命令。"
```

通过：识别到 `github`/`gh-issues` 并给出示例命令。

---

## 5) 当前状态结论

当前环境已进入可长期使用状态：
- 官方版 OpenClaw（非 -cn）运行稳定
- 多 Agent 分工明确
- 技能按角色收敛
- 全局权限已从高权限收回到最小权限路线
- Telegram 自然语言发号施令可用

---

## 6) 后续建议（不影响现在使用）

1. 再补一条网关防爆破：
```bash
openclaw config set gateway.auth.rateLimit '{"maxAttempts":10,"windowMs":60000,"lockoutMs":300000}' --strict-json
openclaw gateway restart
```

2. 每次新增 agent 后，先 `openclaw config get agents.list --json` 再改索引，避免 skills 写错人。

3. 若遇到“completed 但空输出”，继续沿用 main 的 Hard Output Contract + Source Guardrail。

---

记录时间：2026-04-13
阶段标签：phase-2 hardening
