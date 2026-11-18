---
type: 判据
domain: D-提权上线维持
status: 待实测
created: 2026-11-18
tags: [Windows, 提权, 令牌, ACL]
---

# D02 · Windows 权限判据

> 这一篇回答一个问题：**`Access is denied` 到底是谁挡的。**
> 来源：Career 原《必学清单》第 1 项（2026-11-18 拆分后迁入，判据是库主的）。
> **状态：全是判据，一条实测都没有。** 每条后面要接一次靶场验证才算数。

---

## 一、五条判据（缺一条，提权路径就判错）

| 判据 | 它决定什么 |
|---|---|
| **令牌**：主令牌 vs 模拟令牌；UAC 下管理员组拿到的是 **filtered token**（Medium） | "我是管理员"和"我有管理员权限"是两件事 |
| **完整性级别** Medium / High / System | DACL 通过了也未必能写——这是第二道门 |
| **DACL 求值**：拒绝优先于允许；组继承；所有者永远能改权限 | `Access is denied` 到底是哪个 ACE 挡的 |
| **六个特权**：`SeDebug`（读任意进程）、`SeImpersonate`（借别人身份）、`SeBackup`/`SeRestore`（绕过 ACL 读写）、`SeTakeOwnership`、`SeLoadDriver` | `whoami /priv` 里那一行到底能不能用 |
| **服务与进程身份**：服务账户是谁、`sc qc` 该看哪几行 | 服务劫持报得对不对 |

## 二、六条必须能背出来的链条

不是背名词，是能当场推：

1. `whoami` 只给名字，`whoami /all` 才给 privileges 和 groups——**后者才决定走哪条提权路**
2. 管理员登录 UAC 后拿到两个令牌，进程用那个 filtered 的 → 提权要先想办法拿到 unfiltered
3. DACL 拒绝优先：**只要有一条 Deny 命中，后面的 Allow 全不看**
4. `SeImpersonatePrivilege` 本身不等于能提权，还缺一个"能强制 SYSTEM 向你认证的通道"（Potato 家族就是不同通道）
5. `SeBackup`/`SeRestore` 绕过的是**文件 ACL**，不是完整性级别
6. 服务可写 = 换掉它的二进制或改它的路径 → 以服务账户身份执行

## 三、验收（做不到就是没学会）

给 `whoami /priv` + `systeminfo` + 服务列表，**30 分钟排出 3 条提权路径**，按成功率排序，说出每条的前提失效点。

> 自评 Q3 曾把 Windows 特权答成 Linux 的脏牛/SUID——这一篇存在的理由就是补那个洞。见 [[Security/01-能力自评]]。

## 四、不学

进程调度、内存管理、文件系统实现、驱动开发。这些对提权动作没有影响。

---

## 相关

- [[D-提权上线维持/00-索引|D 域索引]]
- [[权限提升]]（Linux 侧底稿）
- [[落地后前10分钟-自查清单]]
- [[Security/01-能力自评|自评 Q3]]
