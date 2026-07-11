
## 📁 准备工作

略

---

## 🔑 配置 Secrets

在仓库 `Settings` → `Secrets and variables` → `Actions` → **Secrets** 中添加以下两个 Secrets：

| Secret 名称 | 说明 |
|-------------|------|
| `GH_TOKEN` | GitHub Personal Access Token，需具有 `repo` 权限，用于自动写入 Variables（首次运行自动创建） |
| `CLOUDFLARE_TUNNEL_TOKEN` | Cloudflare Tunnel Token，从 Cloudflare Zero Trust 面板获取 |

---

## 📝 配置 Variables（可选）

在 **Variables** 标签下，您可以预先设置以下变量（若未设置，首次运行会自动生成并写入 `WARP_REG_JSON` 和 `WARP_CONF_JSON`）：

| Variable 名称 | 说明 | 默认值 |
|---------------|------|--------|
| `WARP_REG_JSON` | 压缩的 `reg.json` 内容（含私钥），首次运行后自动填充 | 空（首次自动生成） |
| `WARP_CONF_JSON` | 压缩的 `conf.json` 内容（含端点、IP），首次运行后自动填充 | 空（首次自动生成） |
| `XRAY_CLIENTS` | Xray 入站客户端配置（JSON 数组） | `[]` |
| `WARP_ENDPOINT` | 自定义 WARP 端点 IP（若不填则从 `conf.json` 自动获取） | 自动获取 |
| `WARP_MTU` | WireGuard MTU 值 | `1280` |
| `XRAY_RUNTIME` | 工作流运行时长（秒），超时后自动结束 | `300` |

---

## 🚀 首次运行（无 Variables）

1. 进入仓库 `Actions` 页面，选择 `Xray WARP Gateway Test` 工作流，点击 `Run workflow`。
2. 工作流将：
   - 安装并注册 WARP，生成 `reg.json` 和 `conf.json`。
   - 从本地文件读取配置，生成 Xray 配置文件。
   - 启动 Xray 和 Cloudflare Tunnel。
   - 自动将两个 JSON 文件压缩后写入 Variables（`WARP_REG_JSON` 和 `WARP_CONF_JSON`），供后续运行使用。
   - 持续运行 `$XRAY_RUNTIME` 秒（默认 300），实时输出日志。
3. 运行结束后，Variables 已保存，下次运行无需再安装 WARP。

---

## 🔁 后续运行（Variables 已存在）

- 工作流会**跳过** `Install & Register WARP` 步骤，直接从 Variables 读取 JSON 并生成配置。
- 启动时间显著缩短，且不会因缓存过期而重新注册 WARP（永久有效）。
- 若您需要更新 WARP 配置（例如更换节点），可删除 Variables `WARP_REG_JSON` 和 `WARP_CONF_JSON`，再次触发工作流，系统将重新注册并更新变量。

---

## 📊 查看日志

- 工作流执行过程中，`Running` 步骤会实时输出 `xray.log` 和 `cloudflared.log` 的内容。
- 您也可以在 `Actions` 页面点击对应运行记录，查看所有步骤的详细输出。

---

## ⚠️ 注意事项

- **`GH_TOKEN` 权限**：必须具有 `repo` 作用域，否则 `gh variable set` 会失败，Variables 无法自动写入，下次运行将重新安装 WARP。
- **Cloudflare Tunnel**：请确保您的 Tunnel 已配置好转发规则（指向 Xray 监听的端口，例如 443），否则外部流量无法访问。
- **模板文件格式**：确保 `xray/config.template.json` 是合法的 JSON，且所有变量名用双引号包裹，如 `"$WARP_PRIVATE_KEY"`。
- **运行时长**：若 `XRAY_RUNTIME` 设为过小（如 10 秒），可能服务未完全启动即被终止；建议至少 60 秒。

---

## 🛠️ 故障排查

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| 首次运行失败 | `GH_TOKEN` 未设置或无权限 | 添加具有 `repo` 权限的 Token |
| `reg.json` 或 `conf.json` 找不到 | 安装注册步骤未执行（Variables 存在） | 删除 Variables 强制重新注册 |
| Xray 启动失败 | 模板文件格式错误或缺少变量 | 检查 `config.template.json` 语法 |
| 日志显示 `timeout` 错误 | `XRAY_RUNTIME` 包含非数字字符 | 设置为纯数字（如 `300`） |
| Cloudflare Tunnel 连接失败 | Tunnel Token 过期或无效 | 在 Cloudflare 面板重新生成 Token |
