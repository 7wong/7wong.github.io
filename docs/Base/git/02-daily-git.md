---
title: Git 的日常
---

Git 的操作指令非常多，此处仅选一些日常操作。



> 注意：以下所有 `[]` 仅作为划分区域，无实际作用，可无视。

## 查看帮助

所有命令行最重要的当然是查看帮助文档：

```bash
git help
```

结果：

```vim
用法：git [--version] [--help] [-C <路径>] [-c <名称>=<取值>]
           [--exec-path[=<路径>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<路径>] [--work-tree=<路径>] [--namespace=<名称>]
           <命令> [<参数>]

这些是各种场合常见的 Git 命令：

开始一个工作区（参见：git help tutorial）
   clone      克隆仓库到一个新目录
   init       创建一个空的 Git 仓库或重新初始化一个已存在的仓库

在当前变更上工作（参见：git help everyday）
   add        添加文件内容至索引
   mv         移动或重命名一个文件、目录或符号链接
   reset      重置当前 HEAD 到指定状态
   rm         从工作区和索引中删除文件

检查历史和状态（参见：git help revisions）
   bisect     通过二分查找定位引入 bug 的提交
   grep       输出和模式匹配的行
   log        显示提交日志
   show       显示各种类型的对象
   status     显示工作区状态

扩展、标记和调校您的历史记录
   branch     列出、创建或删除分支
   checkout   切换分支或恢复工作区文件
   commit     记录变更到仓库
   diff       显示提交之间、提交和工作区之间等的差异
   merge      合并两个或更多开发历史
   rebase     在另一个分支上重新应用提交
   tag        创建、列出、删除或校验一个 GPG 签名的标签对象

协同（参见：git help workflows）
   fetch      从另外一个仓库下载对象和引用
   pull       获取并整合另外的仓库或一个本地分支
   push       更新远程引用和相关的对象

命令 'git help -a' 和 'git help -g' 显示可用的子命令和一些概念帮助。
查看 'git help <命令>' 或 'git help <概念>' 以获取给定子命令或概念的
帮助。
```



## 重命名文件

总是觉得名字不够优雅，常常去修改：

```bash
git mv readme readme.md
```



## 删除文件

删除文件的操作与重命名文件类似：

```bash
git rm file1
```



## 创建分支

上面有了删除分支，当然也要有创建分支，创建分支往往会基于某个分支来创建：

```bash
# 本地
# 基于当前分支创建
git checkout master
git checkout -b branch1
# 基于 branch2 分支创建
git checkout -b branch1 branch2

# 远程
# 基于 branch1 分支创建（一般会将命名相同，方便 push）
git checkout -b branch1 origin/branch1
```

在这里补充一个想法，想要 `git push` 成功的关键是 `fast-forward` 。



## 查看日志

> 参数可以组合，更强大。

### 当前分支

查看当前分支的日志：

```bash
git log
```

单行显示当前分支：

```bash
git log --oneline
```

当前分支的最近几条日志：

```bash
git log -n2
```

单行显示当前分支的最近几条日志：

```bash
git log -n2 --oneline
```

图形化显示当前分支（更复杂的看所有分支示例）：

```bash
git log --graph
```

### 所有分支

显示所有分支的日志：

```bash
git log --all
```

单行显示所有分支的日志：

```bash
git log --all --oneline
```

所有分支的最近几条日志：

```bash
git log --all -n2
```

单行显示所有分支的最近几条日志：

```bash
git log --all -n4 --oneline
```

单行图形化显示所有分支的最近几条日志：

```bash
git log --all -n20 --oneline --graph
```

### 某一分支

可以指定某个分支：

```bash
git log origin/gh-pages
```

## 修改 commit 信息

commit 总是描述的不合意，需要进行修改：

### 修改最近一次 commit 信息

```bash
git commit --amend
```

### 修改之前的 commit 信息

```bash
# 切换到你需要的分支
git checkout x_branch
# 输出该分支的 commit
git log
# rebase 需要修改的父 hash
git rebase -i [父 hash]

# 将对应的 pick 修改为 reword（r）即可
```

### 将连续的多个 commit 合并

其实与上方的操作一致：

```bash
# 切换到你需要的分支
git checkout x_branch
# 输出该分支的 commit
git log
# rebase 需要修改的父 hash
git rebase -i [父 hash]

# 将连续内容的 pick 修改为 squash（s） 即可
```

### 将间隔的多个 commit 合并

依然需要使用 `rebase` ：

```bash
# 切换到你需要的分支
git checkout x_branch
# 输出该分支的 commit
git log
# rebase 需要修改的父 hash，没有父 hash 时就选最后一个即可
git rebase -i [父 hash]

# 1. 拷贝最后一个 hash 手动添加至编辑器最上方
# 2. 将需要合并的 commit 上移至添加的上方
# 3. 将上移的 pick 修改为 squash（s） 即可
```



## 查看区别

经常需要查看文件发生了哪些变更。尤其是现在电脑屏幕小，总感觉会漏掉点什么。

### 工作区和暂存区

```bash
# 显示所有文件的区别
git diff

# 显示某个（多个）文件的区别
git diff -- xxx xxx2
```

### 暂存区和 HEAD

```bash
git diff --cached
```

### 分支之间

```bash
# 选择你想要对比的两个分支
git diff branch1 branch2

# 前面提及具体对比某个文件加上 `--` 即可
git diff branch1 branch2 -- file1
```

### commit 之间

基本与分支之间一致

```bash
# 选择你想要对比的两个 commit hash
git diff [hash1] [hash2]
```



## 取消

简单记一下：变更工作区用 `git checkout`，变更暂存区用 `git reset` 。

### 取消暂存

```bash
# 取消当前所有暂存
git reset HEAD

# 取消暂存某个（多个）文件
git reset HEAD xxx xxx1
```

### 取消 commit

取消 commit 当然也是需要用到 `git reset` 来切换 HEAD 位置：

```bash
# 找到你想要切换回去的 hash 提交记录
git reset --hard [hash]

# 注意：该指令会导致之前 commit 的内容也消失，如果只是想要修改 commit 信息请参照上方的修改 commit 信息
```

### 取消文件修改

```bash
# 取消所有文件修改
git checkout
# 取消某个（多个）文件修改
git checkout -- xxx xxx1
```



## 储藏

主要是用于暂时不想产生新的提交，并有其余任务需要需要处理的时候。

```bash
# 储藏
git stash

# 查看储藏
git stash list

# 重新应用刚刚的储藏
git stash apply

# 重新应用刚刚的储藏还有另一种形式，不过会导致储藏立即移除
git stash pop

# 重新应用更早的储藏
git stash apply stash@{2}

# 移除储藏
git stash drop stash@{0}
```



## 备份（传输协议）

Git 可以用过两种主要的方式在版本库之间传输数据：“哑（dumb）”协议和”智能（smart）“协议。

### 哑协议

这个操作就跟克隆远程仓库类似：

```bash
# 首先进入你需要备份的文件夹
cd ~/backup

# clone，记得在路径的最后增加 `/.git`
git clone --bare /xxx/xxx/your_project/.git
```

### 智能协议

其余部分与哑协议一致：

```bash
git clone --bare file:///xxx/xxx/your_project/.git
```

### 关联备份（远端）

```bash
# 关联 name 关键字在日后可简化操作
git remote add name file:///xxx/xxx/your_project/name.git

# 推送
git push name
```



## 合并

合并方式往往会使用两种：merge、rebase。

### merge 合并

举个例子，将分支 2 合并到分支 1：

```bash
git checkout branch1
# 本地
git merge branch2
# 远程
git merge origin/branch2

# 合并无关联的分支（即没有共同的祖先）
git merge branch2 --allow-unrelated-histories
```

合并相同的分支：

```bash
# 相同的分支当然是一个本地，一个远程
git checkout branch1

# 不同文件
git merge origin/branch1
# 相同文件不同区域
git merge origin/branch1
# 相同文件相同区域
# 这时 git 无法自动合并，需要手工上了
# 命令行或者图形化界面均可
# 修改完成
git commit -am"resolved the conflict"
# 文件名发生变更(单方修改文件名)
# git 能够自动处理
git merge origin/branch1
# 文件名发生变更（双方/多方修改文件名）
# git 能够识别为修改了文件名，但是冲突需要手动解决
# 去除不需要的文件
git rm file1 file2
# 将需要的文件保存到暂存区并提交 commit
git commit -am"resolved the conflict"
```

### rebase 合并

很多公司并不喜欢多条分支交错的感觉（不够直观），采用 rebase 方式可以使记录保持一条线性。

处理远程：

```bash
# 切换到对应分支
git checkout branch1

# 处理远程分支 branch2
git rebase origin/branch2 

# 解决冲突的文件
vim file1
vim xxx
# 解决完冲突继续 rebase
git rebase --continue
# 继续解决冲突，重复上方 vim 操作
# git rebase 就是这点麻烦，你在 branch1 的 commit 有几次，就会发生几次 rebase，因为 rebase 是把所有的 commit 记录都同步到 master 分支

# 上面解决冲突的方式很麻烦，万一你在 branch1 有十几个 commit 那…
# 所以 git 也提供 Rerere 工具
# 启用工具
git config --global rerere.enabled true
# 先使用 merge 记录操作
git merge origin/branch2
# 提示有冲突需要解决
vim file1
git add .
git commit -am"conflict record"
# 回退处理
git reset --hard HEAD~1
# 继续使用 rebase 来合并
git rebase origin/branch2 
# git 会按照之前的 rerere 的规则自动处理冲突
# 仅需要对比一下文件是否按所想的处理了
vim file1
git add file1
# 继续 rebase
git rebase --continue
# 重复 vim - continue 操作
```

