---
type: 问题
domain: C-执行与驻留
status: 待验证
created: 2026-10-27
tags: [windows, 凭据, 内网]
---

# Windows 上凭据到底藏在哪

> **P0 · 思维转变题。** 自评 Q6 从"定位域控"直接跳到"找漏洞"，整条路径没有一处在问"谁登录过这台机器"。
> **外网是找洞，内网是找人。** 这个问题的答案决定了你能不能从"找洞"切到"找人"。

## 现象

- 拿到一台 Windows 的本地管理员权限后，我的第一反应是扫描和横向。
- 没想过"这台机器上已经有现成的凭据"。

## 我的猜测（先写自己的，再往下做）

> （先猜：一台跑了半年的业务服务器，最可能在哪留下明文或可复用凭据？写下你的前三个猜测）

## 实验设计

环境：Vulnstack 里任意一台已拿到的 Windows 主机。**不许用任何抓取工具先跑**，纯手工翻一遍：

| 位置 | 具体命令/路径 | 记什么 |
|---|---|---|
| 内存 | （最后做）`procdump` / `comsvcs.dll MiniDump` | 有几个账号、有没有域管 |
| IIS 配置 | `C:\Windows\System32\inetsrv\config\applicationHost.config`、站点目录下的 `web.config` | 有没有连接串 |
| 数据库配置文件 | 各站点根目录、`appsettings.json`、`.env` | 明文密码？ |
| 计划任务 | `schtasks /query /fo LIST /v` | 命令里有没有带密码 |
| PowerShell 历史 | `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt` | 有没有人敲过明文口令 |
| 浏览器 | Chrome/Edge 的 `Login Data` | 保存的密码 |
| 服务 | `sc qc <name>` 看 `SERVICE_START_NAME` 和 binPath | 有没有写死密码 |
| 注册表 | `HKLM\...\Winlogon` 的 DefaultPassword | 自动登录密码 |
| 无人值守文件 | `C:\unattend.xml`、`sysprep.inf` | 装机密码 |
| 共享目录 | `net share` + 翻脚本和配置文件 | 运维留的脚本里有没有密码 |

**然后**再抓 lsass，对比：手工翻到的 vs 内存里拿到的，重叠多少？

## 关闭条件

- [ ] 能说出至少 8 个凭据位置，以及各自的**读取前提**（要什么权限）
- [ ] 能解释：**为什么本地管理员的 hash 出不了这台机器**（除非密码复用）
- [ ] 能说出"这台机器上有没有域管登录过"该怎么判断（不只是抓 lsass）
- [ ] 形成一条自己的优先级顺序：什么环境先翻哪、为什么

## 关联

- [[01-能力自评]] · Q1（遗漏点）· Q6（跳过环节）
- 产出 → `C-执行与驻留/落地后前10分钟-自查清单.md`
