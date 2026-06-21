工作区 - 暂存区 - 版本库 - 远程仓库

### 命令行

初始化仓库 `git init`

查看当前仓库状态 `git status`

添加单个文件 `git add hello.txt`

添加当前目录下所有文件（常用）`git add .`

添加所有已修改的文件（不包括新创建的文件）`git add -u`

提交到版本库 (`git commit -m '说明'`) 

查看提交历史`git log`

查看简洁历史（只显示版本ID和说明）`git log --oneline`

找不同 `git diff`

--- `git diff` : 查看还未添加到暂存区（未 git add）的修改

--- `git diff --staged`|`git diff --cached` : 查看已暂存（已 git add）但还未提交的修改

--- `git diff HEAD` : 查看“从上次提交以来的所有修改”

--- `git diff abc123 def456` : 比较两个具体的提交

--- `git diff main feature-branch` : 比较当前分支和另一个分支

--- `git diff HEAD~1 HEAD` : 比较当前分支和它的前一个提交



### **常规流程**

1.  提交文件至暂存区 - `git add .`
2.  将暂存区的文件提交到版本库 - `git commit -m '...'`
3.  将远程更新拉取到本地并变基 - `git pull --rebase origin 分支名`
4.  推送到远程仓库的同名分支 - `git push origin 分支名`



### **关于撤销**

1. 尚未暂存，取消工作区的修改：`git checkout -- .`  或  `git checkout -- '文件名'`
2. 撤销已暂存的修改：
    * 将暂存区和工作区都重置到当前最新提交（HEAD），丢弃所有本地修改: `git reset --hard HEAD`
    * 取消暂存区的提交，保留工作区的修改:  `git reset` 
    * 仅将指定文件移出暂存区，其他已暂存文件不受影响。工作区中该文件的修改同样会保留:  `git reset HEAD <文件名>` 
3. 撤销已提交的修改
    * 彻底丢弃上次的提交和所有相关修改： `git reset --hard HEAD~1`
    * 仅撤销提交，但保留工作区文件的修改内容:  `git reset --soft HEAD~1`
4. 撤销已推送的修改：`git revert 推送id`



### 关于分支

- `git checkout -b 分支名` 创建并切换到新分支
- `git push origin develop` 只推送，后续推送都要执行`git push origin develop`
- `git push -u origin develop` 推送并建立关联，后续只需要执行`git push`
- `git branch` 查看分支
- `git pull、git fetch、git merge` 一次性获取并合并、查看远程更新、合并
- `git branch -d 分支名` 删除分支
- `git branch -D 分支名` 强制删除分支
- `git branch -m 分支名` 重命名分支
- `git branch -M 分支名` 强制重命名分支
- `git branch -r` 查看远程分支
- `git branch -a` 查看本地和远程分支
- `git branch -vv` 查看本地分支与远程分支的关联关系



### 关于变基（只能在推送远程仓库前变基）

一. 整理本地分支

git checkout feature

git rebase main

二. 修改提交信息

多次提交合并成单次提交

```
git rebase -i HEAD~提交次数
执行后↓↓↓↓↓↓↓↓↓↓↓↓↓

pick 3e4b5c2 feat: 添加功能A
s 9d2a7f1 feat: 添加工能B
s 1b4c5e6 fix: 修复功能B的拼写
s 7a3f8d2 debug: 临时调试

# 变基 a1b2c3d..7a3f8d2 到 a1b2c3d（4 个提交）
#
# 命令:
# p, pick <提交> = 使用提交
# r, reword <提交> = 使用提交，但修改提交信息
# e, edit <提交> = 使用提交，但停止以修改提交
# s, squash <提交> = 使用提交，但合并到前一个提交
# f, fixup <提交> = 类似于 "squash"，但丢弃提交信息
# d, drop <提交> = 删除提交
#
# 这些行可以被重新排序；它们会从上到下依次执行。
```

```
vim 编辑命令
按 Esc 键（退出编辑模式，回到命令模式）
按 i 键（回到编辑模式）
按 :wq + Enter 键（保存并退出）
按 :q! + Enter 键（保存并退出）
```



### 关于存储(stash)

```
git stash                      存。将工作区和暂存区打包入栈。快捷命令
git stash push -m "描述"        存。将工作区和暂存区打包入栈。推荐用 push替代老旧的 save
git stash list                 看。列出所有储藏记录（stash@{0}是最新的）。
git stash show -p stash@{0}    查。-p查看具体改了哪些代码（Diff）。
git stash pop                  取+删。应用最新的储藏（stash@{0}）并删除该记录。最常用。
git stash apply                取不删。应用储藏，但记录还留在栈里。可重复应用。
git stash drop stash@{1}       删。删除指定的储藏记录（不恢复内容）。
git stash clear                清空。删除所有储藏。危险操作，不可逆。
```



### 关于Cherry-pick

将某个提交的**变更（diff）** 在当前分支重放一次，生成一个**新的提交**（哈希值不同）

```
git cherry-pick <commit-hash>
-n 应用变更，但不提交(想手动调整代码后再提交)
-e 允许编辑提交信息(想修改原提交的说明文字)
-x 在原提交信息中追加 (cherry picked from commit ...)(审计追踪，记录来源)
-s 在提交信息末尾添加签名行(公司要求签署提交)
--abort 放弃 cherry-pick，回到操作前状态(冲突太多，想放弃重来)
--continue 解决冲突后继续 cherry-pick
```

