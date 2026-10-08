# AgentFlow 发布产物

本仓库是 AgentFlow 的**分发仓**：只放安装包与更新产物，**不含源码**（源码仓是私有的）。
本仓内容由 `packaging/release.py` 自动推送，**请勿手工改动产物文件**（会被下次发版覆盖）。

---

## 我该下载什么

| 文件 | 给谁用 | 说明 |
| --- | --- | --- |
| `AgentFlow-Setup-<版本>.exe` | **新用户** | 安装包，约 69 MB。含内置运行时与前端产物，装完即可用 |
| `app-<版本>.zip` | 已装用户（程序自动取） | 应用层增量包，约 650 KB。**不用手动下载** |
| `manifest.json` | 程序自动读 | 更新清单：最新版本号 + 增量包地址 + sha256 + 本版更新内容 |

> 下载入口：本仓右侧 **Releases**（每个版本一个 tag，资产在该版本的 Release 里）。
> 仓根只常驻 `manifest.json`（更新入口）与这份 README；历史安装包不堆在仓根，去对应 Release 找。

---

## 新用户：安装

1. 打开本仓 **Releases**，下载**最新版本**的 `AgentFlow-Setup-<版本>.exe`
2. 双击运行 —— 安装是**当前用户级**的，不需要管理员权限
3. 装完自动打开控制台，按首次配置向导填入模型 API Key 即可开工

- **提示「Windows 已保护你的电脑」？** 安装包未做代码签名，属预期现象。点「更多信息」→「仍要运行」。
- 安装目录默认 `%LOCALAPPDATA%\AgentFlow`；开始菜单会有「AgentFlow 控制台」与「卸载 AgentFlow」。
- 需要 Git Bash（平台默认执行壳）时无需自备：安装包内置 MinGit，系统已有 git 则优先用系统的。

## 已装用户：更新

**通常不需要手动操作**：程序启动后自行检查更新，控制台顶栏出现「⬆ 有新版本」时点一下即可。
更新**只替换程序文件**，你的配置、API Key、数据库、任务历史都在 `%USERPROFILE%\.agentflow`，不会被动。

两种情况需要手动装一次新安装包：

1. 更新页提示**「需重装安装包」** —— 该版本调整了内置依赖（增量包换不了第三方依赖）。
2. 有远端时提示「待推送」推不动 —— 网络/凭据问题，先到 **设置 → Git** 页签处理。

## 版本与校验

- 每个版本的更新内容（提交清单）写在对应 Release 说明里，也随 `manifest.json` 的 `notes` 字段下发
  （控制台「设置 → 更新」页展示的就是它）。
- 增量包在 `manifest.json` 里带 `sha256`，程序下载后会校验；手动校验：

  ```powershell
  Get-FileHash .\app-<版本>.zip -Algorithm SHA256
  ```

- `manifest.json` 字段：`version`（最新版）· `app_zip.{name,url,sha256}`（增量包）·
  `requires_runtime`（对不上就必须重装安装包）· `commit` / `notes`（源码提交与更新内容）。

## 国内网络说明

GitHub 直连与 raw 直连在国内多不可达，本项目的更新链路默认走镜像前缀（`ghproxy.net`）：

```
manifest 入口：https://ghproxy.net/https://raw.githubusercontent.com/Gexomaikoine/AgentFlow/main/manifest.json
增量包地址  ：https://ghproxy.net/https://github.com/Gexomaikoine/AgentFlow/releases/download/v<版本>/app-<版本>.zip
```

镜像不可用时可在 **设置 → 更新** 里改 `manifest_url`（可填直连地址）。

## 卸载

开始菜单运行「卸载 AgentFlow」，或「设置 → 应用」里卸载。

卸载**不会删除**数据目录 `%USERPROFILE%\.agentflow`（配置、密钥、数据库、历史记录都在那），
要彻底清理请手动删除该目录。注意：`~/.agentflow/.env` 里存着你的模型 API Key。

## 常见问题

- **装完打不开 / 控制台空白**：先看 `%USERPROFILE%\.agentflow\serve.log`；多数是端口 8765 被占用。
- **想回滚旧版本**：程序更新前会在 `%LOCALAPPDATA%\AgentFlow` 旁保留上一份 `app/`，更新失败会
  自动回滚；手动回滚可到 Releases 下载旧版安装包覆盖安装。
- **换了机器**：装最新安装包后，把 `~/.agentflow` 目录整体拷过去即可带回全部配置与历史。

---

维护者：发布用 `DIST_REPO_TOKEN=<pat> python packaging/release.py --publish repo --dist-repo Gexomaikoine/AgentFlow`
（CI 由推 tag 自动触发；详细配置见源码仓 `.github/workflows/release.yml` 顶部说明）。
