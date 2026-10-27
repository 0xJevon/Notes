---
type: 问题
domain: D-提权
status: 待验证
created: 2026-10-27
tags: [windows, 提权, potato]
---

# SeImpersonatePrivilege 怎么变成 SYSTEM

> **P0 · 真实缺口。** 自评 Q3 整题空白。

## 现象

`whoami /priv` 显示 `SeImpersonatePrivilege  Enabled`。
我知道这是"能提权"的信号，但不知道具体怎么做，也不知道什么情况下做了没用。

## 我的猜测（先写自己的，再往下做）

> （提示：这个特权字面意思是"模拟"，那"模拟谁"？谁会把身份借给你？）

## 实验设计

环境：Vulnstack 里从 **IIS 应用池账号**出发（不是管理员）。

1. `whoami /priv` 确认有 SeImpersonate
2. 先查系统版本：`systeminfo | findstr /i "OS Name OS Version"`
3. 按版本选工具，**每次只换一个变量**：

| 尝试 | 记录什么 |
|---|---|
| PrintSpoofer | 成功/失败，Spooler 服务状态（`sc query spooler`） |
| Juicy Potato | 在 2016 和 2019 上分别跑，看差异 |
| GodPotato | 同上 |

4. 关键实验：**关掉 Spooler 服务**再跑 PrintSpoofer，看是否失败——用来验证"通道"这个前提

## 关闭条件

- [ ] 能用一句话说清原理：**强制一个 SYSTEM 进程向我控制的通道认证 → 我模拟它的令牌**
- [ ] 能说出「有特权 ≠ 能提权」缺的那一环是什么（通道）
- [ ] 能按系统版本判断哪个 Potato 家族成员失效、为什么失效
- [ ] **能说出这套全部失效时的替代路径**（令牌窃取 / 服务配置错误 / AlwaysInstallElevated）

## 关联

- [[01-能力自评]] · Q3
- 前置 → [[Windows的令牌和特权是什么]]
