### 首次推送

```bash
cd /path/to/Notes
git init
git add .
git commit -m "Initial commit: 添加笔记"
git branch -M main

# 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/你的用户名/Notes.git
# 或SSH方式：
# git remote add origin git@github.com:你的用户名/Notes.git

git push -u origin main
```

### 📦 基础命令

| 命令                    | 作用                                       |
| :---------------------- | :----------------------------------------- |
| git init                | 初始化一个 Git 仓库（创建 .git 文件夹）。  |
| git clone <url>         | 克隆远程仓库到本地。                       |
| git status              | 查看当前工作目录状态（文件修改、暂存等）。 |
| git add <file>          | 添加文件到暂存区。                         |
| git commit -m "message" | 提交暂存区文件到本地仓库，并添加提交信息。 |
| git log                 | 查看提交历史。                             |

------

### 🌱 分支管理

| 命令                        | 作用                                 |
| :-------------------------- | :----------------------------------- |
| git branch                  | 查看本地分支列表。                   |
| git branch <branch-name>    | 创建一个新分支。                     |
| git checkout <branch-name>  | 切换到指定分支。                     |
| git switch <branch-name>    | 切换到指定分支（推荐使用的新命令）。 |
| git merge <branch-name>     | 合并指定分支到当前分支。             |
| git branch -d <branch-name> | 删除已合并的分支。                   |
| git branch -D <branch-name> | 强制删除分支。                       |

------

### 🌍 远程仓库操作

| 命令                             | 作用                                 |
| :------------------------------- | :----------------------------------- |
| git remote -v                    | 查看远程仓库地址。                   |
| git remote add <name> <url>      | 添加远程仓库。                       |
| git pull                         | 从远程仓库拉取代码并合并。           |
| git push                         | 将当前分支代码推送到远程仓库。       |
| git push -u origin <branch-name> | 推送分支并设置上游（跟踪）分支。     |
| git fetch                        | 获取远程仓库最新数据，但不自动合并。 |
| git remote remove <name>         | 删除远程仓库连接。                   |

------

### 🔍 查看与对比

| 命令                 | 作用                         |
| :------------------- | :--------------------------- |
| git diff             | 查看工作区未暂存的更改。     |
| git diff --staged    | 查看暂存区的更改。           |
| git log --oneline    | 简洁地查看提交历史。         |
| git show <commit-id> | 查看某次提交的详细信息。     |
| git blame <file>     | 查看文件每行对应的提交记录。 |

------

### ♻️ 撤销与回退

| 命令                          | 作用                                 |
| :---------------------------- | :----------------------------------- |
| git checkout -- <file>        | 撤销对工作区文件的修改。             |
| git restore <file>            | 撤销文件修改（新命令）。             |
| git reset HEAD <file>         | 将文件从暂存区移回工作区。           |
| git reset --soft <commit-id>  | 回退到某提交，保留所有改动。         |
| git reset --mixed <commit-id> | 回退并清除暂存区，但保留工作区改动。 |
| git reset --hard <commit-id>  | 回退并清除所有改动。                 |
| git revert <commit-id>        | 生成一个新提交，撤销某次提交内容。   |

------

### 🏷️ 标签管理

| 命令                       | 作用                 |
| :------------------------- | :------------------- |
| git tag                    | 查看所有标签。       |
| git tag <tag-name>         | 创建标签。           |
| git tag -d <tag-name>      | 删除标签。           |
| git push origin <tag-name> | 推送指定标签。       |
| git push origin --tags     | 推送所有标签到远程。 |

------

### 🗃️ Stash（临时保存）

| 命令            | 作用                       |
| :-------------- | :------------------------- |
| git stash       | 暂存当前未提交的修改。     |
| git stash list  | 查看所有暂存记录。         |
| git stash apply | 恢复最近一次 stash 内容。  |
| git stash drop  | 删除最近一次 stash。       |
| git stash pop   | 恢复并删除最近一次 stash。 |

------

### 🛠️ 其他实用命令

| 命令                                             | 作用                         |
| :----------------------------------------------- | :--------------------------- |
| git config --global user.name "Your Name"        | 设置全局用户名。             |
| git config --global user.email "you@example.com" | 设置全局邮箱。               |
| git config --list                                | 查看当前 Git 配置。          |
| git clean -f                                     | 删除未被跟踪的文件。         |
| git cherry-pick <commit-id>                      | 应用某个提交到当前分支。     |
| git rebase <branch>                              | 当前分支变基到目标分支之上。 |