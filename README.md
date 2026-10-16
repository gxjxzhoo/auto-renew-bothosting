## 🚀 Bot-hosting 自动续期（GitHub Actions）

这是一个基于 GitHub Actions 的自动化脚本，用于定时登录自动续期 [Bot-hosting](https://bot-hosting.net) 服务。

⚠️ 有cf盾,太垃圾的机房节点可能过不了，建议用稍微干净点的节点,[B2proxy住宅代理](https://www.b2proxy.com/signup?code=0F5133)

━━━━━━━━━━━━━━━━━━━━━━

### 🔐 Secrets 配置说明

| Secret 名称         | 是否必填 | 说明                                              |
|---------------------|----------|---------------------------------------------------|
| EMAIL              | ❌ 可选  | 仅单账号时用于通知显示；多账号时忽略，自动读取真实邮箱 |
| SESSION_TOKEN      | ❌ 可选  | Bot-hosting session_token，cookie里获取               |
| DISCORD_TOKEN      | ✅ 必填  | Discord Token，SESSION_TOKEN失效时自动OAuth登录        |
| GH_TOKEN           | ❌ 可选  | GitHub Fine-grained token，仅需本仓库 Secrets 读写权限，用于自动更新 session_token |
| SERVER_URL         | ❌ 可选  | 服务页 /a/d/<uuid>，多账号时一行一个按行号配对；不填自动发现 |
| NODE_LINK          | ❌ 可选  | 代理链接（如 vless:// vmess:// trojan:// hysteria2:// tuic:// anytls:// socks5:// )|
| TG_BOT_TOKEN       | ❌ 可选  | Telegram Bot Token（用于发送通知）                      |
| TG_CHAT_ID         | ❌ 可选  | Telegram Chat ID（接收通知的用户或群组 ID）               |

### 多账号（v4.0+）
`SESSION_TOKEN` 和 `DISCORD_TOKEN` 里一行写一个账号，按行号配对（第 1 行配第 1 行就是账号 1，以此类推）：

```
# SESSION_TOKEN
账号1的session_token
账号2的session_token
账号3的session_token
```
```
# DISCORD_TOKEN
账号1的discord_token
账号2的discord_token
账号3的discord_token
```
- `DISCORD_TOKEN` 每行仍支持 `备注,token` 写法（备注只用于日志区分账号）
- 每个账号用独立浏览器跑，cookie 不串号；单个账号失败不影响其他账号
- 跑完后所有账号刷新掉的 token 一次性按行回写到 `SESSION_TOKEN` Secret（需配 `GH_TOKEN`）
- 两边行数对不上会在日志里大声告警；只写一行就是单账号，和老版本完全兼容
- v4.1：空行保留占位、行号永不错位；某行无 cookie 时直接走 Discord 登录，登上就回写 cookie；Discord 也失败则该行写 `FAILED_xxxx` 占位符；同一账号一轮只处理一次，重复自动跳过

### v4.2 更新
- 修复：多账号且设置了 `EMAIL` 时，第 2 个起的账号全部被误判为重复跳过
- 修复：续期按钮的「可用/冷却」判断与实际点击的不是同一个元素
- 修复：DD/MM/YYYY 日期被误读
- 新增：`SERVER_URL` 支持多账号按行配对
- 安全：代理脚本内置；checkout 不保留凭据；截图仅私有仓库上传；日志里出口 IP 脱敏；依赖锁定版本
- 新增：工作流保活

━━━━━━━━━━━━━━━━━━━━━━

## 部署步骤
1：fork 本项目，在actions菜单允许工作流

2：在`setting`➡`secrets and variables`➡`Actions` 里添加上方必填的secrets

3：去actions菜单手动试运行工作流,根据自己的服务到期日期自行在[renew.yml](.github/workflows/renew.yml)里调整cron运行时间

### SESSION_TOKEN 获取
登录你的账号,按F12或页面空白处 右键➡检查➡选择应用程序或appcations 找到对应的字段点击获取对应的值，详情如图
<img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/e532b0d6-9f12-45fd-8af9-69e1029a1a92" />

### DISCORD_TOKEN 获取（用于 SESSION_TOKEN 失效后备用登录
1. 浏览器登录 Discord（网页版）
2. 按 F12 打开开发者工具 ➡ 网络 ➡ 点击任意频道 ➡ 选择左侧的任意api 
3. 找到名为 `authorization字段` 的 值，即为discord token,详情如图所示
<img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/7276d62d-31ff-452c-9e13-165af8323f53" />


> **作用**：当 `SESSION_TOKEN` 过期导致登录失败时，脚本会自动使用 Discord Token 走 OAuth 流程重新登录，并自动更新 `SESSION_TOKEN` Secret，实现永久免维护。


### 获取 `GH_TOKEN`（Fine-grained，最小权限）
1. GitHub 右上角头像 → Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token
2. Expiration：建议 90 天或 1 年（不要选永不过期），到期前重新生成即可
3. Repository access：选 **Only select repositories**，只勾你 fork 的这个仓库
4. Permissions → Repository permissions → **Secrets: Read and write**，其余保持 No access
5. 生成后复制保存到本仓库的 `GH_TOKEN` Secret

> ⚠️ 不要使用「全部勾选 + 永不过期」的 classic token，泄露后等于整个 GitHub 账号被接管。

## 注意事项
* 必填变量必须要填写
* NODE_LINK支持的代理协议有：vmess,vless,hysteria2,tuic,anytls,socks5等
* 自动续期不代表可以无底线的薅羊毛,不建议多账号
* cron运行时间不一定准确,得根据实际到期时间修改,可在设置里暂停actions功能再开启
* 代理脚本 `setup_proxy.sh` 已放在仓库内，不再在线下载执行
* 调试截图只在**私有仓库**上传（公开仓库的 artifact 任何人可下载）；建议把仓库设为私有
* 工作流每次运行会自动刷新启用状态，避免 60 天无提交被 GitHub 停用
* Discord token 拥有你 Discord 账号的完整权限，自动化使用有被封号的风险，请自行评估

## ⚠️ 免责声明
* 本程序仅供学习了解, 非盈利目的，如转载须注明来源。
* 使用本程序必循遵守部署服务器所在地、所在国家和用户所在国家的法律法规, 程序作者不对使用者任何不当行为负责。
