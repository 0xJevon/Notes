---
type: 问题
domain: D-提权
status: 待验证
created: 2026-10-27
tags: [windows, 权限模型]
---

# Windows 的令牌和特权到底是什么

> **P0 · 结构性缺口。** 自评 Q3 问 Windows 特权，答成了脏牛和 SUID。
> 这不是"少背一个知识点"，是**整套 Windows 平台认知缺失**。这个不补，后面的提权、横向、域渗透全部建立不起来。

## 现象

- 我的提权知识全是 Linux 侧：内核漏洞、SUID、定时任务。
- `whoami /priv` 里那串 `SeXxxPrivilege` 我不知道该怎么用。

## 我的猜测（先写自己的，再往下做）

> （对比一下：Linux 的 SUID 是什么？它和 Windows 的"特权"是一回事吗？）

## 实验设计

同一台 Windows 上，用**三种不同身份**分别跑 `whoami /all`，把输出存下来对比：

| 身份 | 怎么获得 |
|---|---|
| IIS 应用池账号 | webshell 默认就是这个 |
| 普通本地用户 | `net user test /add` |
| SYSTEM | psexec -s 或计划任务以 SYSTEM 运行 |

逐个对比三样东西：
1. **组成员**（`BUILTIN\Users` / `Administrators` / `IIS_IUSRS`）—— 决定你能读什么文件
2. **特权列表**（`SeImpersonate` / `SeDebug` / `SeBackupPrivilege`）—— 决定你能做哪些"越权动作"
3. **完整性级别**（`whoami /groups` 里的 Mandatory Label）—— 决定你能不能写某些进程

## 关闭条件

- [ ] 能解释：**为什么 IIS 应用池账号有 `SeImpersonatePrivilege`，而普通用户没有**
- [ ] 能说清"组"和"特权"是两套东西：组管**资源访问**，特权管**系统动作**
- [ ] 能列举 3 个高价值特权（`SeImpersonate` / `SeDebug` / `SeBackup`）各自能干什么
- [ ] 能说出 Linux SUID 与 Windows 特权的本质差异（一个是文件属性，一个是进程令牌里的位）

## 关联

- [[01-能力自评]] · Q3
- 后续 → [[SeImpersonatePrivilege怎么变成SYSTEM]]
