# OpenClaw 部署与全自动化运维完整实录

本日志详尽记录了在 NAS 环境下从零部署 OpenClaw 2026.4.2 版本的完整路径，涵盖了网络边界突破、路径陷阱修复、自动化脚本以及权限重构过程。

---

## 1. 基础环境
- **部署身份**: root
- **操作系统**: 飞牛私有云 (NAS)
- **核心版本**: OpenClaw 2026.4.2 (d74a122)
- **网关地址**: 192.168.31.50

---

## 2. 局域网访问与零信任边界突破 (关键配置)
默认情况下，网关仅允许本机环回地址访问。为了在局域网 PC 或异地组网环境下控制网关，进行了以下重构：
### 2.1 解除 CORS 限制
修改 `~/.openclaw/openclaw.json`，在 `gateway.controlUi` 节点添加访问来源：
```json
"allowedOrigins": [
  "http://localhost:18789",
  "http://127.0.0.1:18789",
  "http://192.168.31.50:18789"
]
```
*(注：上方的反引号在实际写入时会自动格式化)*

### 2.2 零信任设备授权 (解决 1008 报错)
外部 IP 首次连接时请求会被挂起，需在宿主机终端执行信任注入：
- 查询挂起请求：`openclaw devices list`
- 放行指定设备：`openclaw devices approve <requestId>`

---

## 3. 关键故障：/root 目录执行陷阱
### 现象与偏差
在初期批量安装技能时，因直接在 `/root` 目录下运行命令，技能被错误下载至 `/root/skills`，导致 `openclaw skills list` 查询为空（网关仅识别 `workspace/skills`）。
### 物理修复动作
1. **纠偏**: `mkdir -p ~/.openclaw/workspace/skills`
2. **迁移**: `mv /root/skills/* ~/.openclaw/workspace/skills/`
3. **清理**: `rm -rf /root/skills` 防止机器人逻辑混淆。

---

## 4. 自动化交互 (Expect 脚本应用)
针对安装过程中的 TUI（图形界面）拦截（如 `Install anyway?`），引入 `expect` 自动化处理：
- **依赖**: `apt-get install expect -y`
- **逻辑**: 自动捕获 `[y/N]` 或 `continue?` 信号并发送 `\r`，实现无人值守安装。

---

## 5. 权限体系彻底重构
为了实现"完全自主干活"且无须人工在 Telegram 点击确认，执行了以下全局配置：
- **能力释放**: `openclaw config set tools.profile full`
- **宿主穿透**: `openclaw config set tools.exec.host gateway` (允许读写 NAS 文件)
- **审批关闭**: 将 `approvals.exec`, `approvals.skill`, `approvals.plugin`, `approvals.node` 的 `enabled` 全部设为 `false`。

---

## 6. GitHub 身份识别修正 (Git Push 报错)
在尝试通过 Bot 推送时遇到"无法获取用户名"的报错，通过以下步骤闭环：
- **双变量注入**: 在 `~/.openclaw/.env` 中不仅写入 `GITHUB_TOKEN`，还必须追加 `GITHUB_USER=dengfhqqq`。
- **Git 全局签名**: 执行 `git config --global user.name "dengfhqqq"` 和对应的 `user.email`。

---
*记录人：dengfhqqq*
*最后更新：2026-04-03*
