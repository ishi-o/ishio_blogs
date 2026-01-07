---
title: 'Git: 使用git进行版本管理'
date: 2024-06-12T00:00:00.000Z
categories:
  - Tools & Utilities
  - manager
  - version manager
tags:
  - beginner
  - git
---
<!-- placeholder -->
<!-- more -->

# 使用`Git`进行版本管理

## 介绍

- `Git`是一个分布式版本管理工具，它诞生的故事很有意思->[Git的历史](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-Git-%E7%AE%80%E5%8F%B2)
- 版本控制器：当我们需要修改代码但又担心无法回退时，我们需要创建副本。当修改次数多、修改规模大时，我们很难记住这些副本实现了什么功能或是修复了哪些`bug`，也不知道改动了哪里，而这只是一人修改的情况，如果是一个需要互相沟通的团队呢？因此，版本控制器应运而生，它方便我们**查看历史版本、快速回退、创建版本分支**等。总而言之，一个合格的程序员应学会熟练使用`Git`这个高效强大且开源的版本控制工具

## 基本概念

- `Git`是分布式版本控制系统，即除了中心仓库以外，每个开发者在本地有完整的仓库副本
- `Git`在本地的工作目录中，通过`.git`目录存储所有的数据
- `Git`在本地分为三个区域
  - 工作区：当前、即时的内容，但修改没有保存在`.git`里
  - 暂存区：通过`git add`保存的修改会被保存到暂存区，即`.git`里

    暂存区又被称为`index`

  - 本地仓库：通过`git commit`将暂存区的修改提交到本地仓库

    当前指向的提交能通过`HEAD`指针访问

- 文件的状态：

  ```
  ?? file.txt        # 未跟踪 (Untracked)
  A  file.txt        # 新增到暂存区 (Added to index)
   M file.txt        # 已修改，未暂存 (M前有空格)
  M  file.txt        # 已修改，已暂存
  MM file.txt        # 已修改，暂存后又修改了
  D  file.txt        # 已删除，已暂存
   D file.txt        # 已删除，未暂存 (D前有空格)
  R  old -> new      # 重命名
  C  file.txt        # 复制
  UU file.txt        # 冲突 (Unmerged)
  ```

- `Git`会将一次提交生成一个哈希值来标记，并使用**`HEAD`**来指向当前所在的提交
- `Git`将新增行和删除行都视为文件的修改，即视为`M`

  将修改行视为删除行和新增行的组合

- `Git`还支持一个文件内的部分修改块进行单独暂存，这一部分修改默认使用`Myers`算法求得，结果被称为`hunk`
- `Git`提供分支管理，分支名称指向该分支的最新提交，就像`HEAD`指向当前所在提交类似的
- `Git`对远程仓库在本地的跟踪：支持使用自定义名称指向，例如`origin`和`upstream`

  注意这仍是在本地的，只不过是跟踪了远程仓库，它们不会自动更新

  可以通过`origin/xxx`访问本地存储的`origin`仓库的`xxx`分支

## 命令行符号

- `--`：用于消除歧义，在`--`后跟文件名不会有歧义
- `~x`：相对引用，`HEAD~x`指向`HEAD`之前的沿着主分支的第`x`代父提交，`x`表示向上回溯的代数

  `HEAD`等价于`HEAD~0`、`HEAD~`等价于`HEAD~1`，`~`通常用于`HEAD`，但是也完全支持对分支名使用，支持链式调用`HEAD~1~1`

  ```
            B---C---D [branch1] J [branch2]
           /         \         / \
  [main]  A---E---F---G---H---I---K
  ```

  例如对于`K`这类合并提交，`K~1`指向`I`、`K~2`等价于`I~1`(非合并提交)因此指向`H`、`K~4`等价于`G~1`指向`F`

- `^x`：相对引用，`HEAD~x`指向`HEAD`之前的第`x`个**父提交**

  `HEAD^`等价于`HEAD^1`等价于`HEAD~1`、`HEAD^^`等价于`HEAD^2`，支持混合的链式调用`HEAD^~`

  同样是上面的例子，`K^1`指向`I`、`K^2`指向`J`

- `@{x}`：引用日志的相对引用，表示某分支在`x`次操作之前的位置，即当前指向的是`HEAD@{0}`

  可以通过`git reflog`查看引用日志

- `:!<pattern>`：用于排除`<pattern>`匹配的文件
- `A..B`：两点语法，用于查看`B`相对于`A`不同的地方，可用于`diff`、`log`等子命令

  哈希值、`HEAD`、分支名（包括和远程仓库关联的分支）都可以

  默认，`A`或`B`省略时，使用`HEAD`

- `A...B`：三点语法，用于查看`A`和`B`相对于`LCA(A, B)`不同的地方，可用于`diff`、`log`等子命令

## 基本配置(`config`)

- `git config`子命令用于修改**项目配置**，也可以通过修改`.git/config`文件
- `git config --global [xxx]`：修改**全局配置**，文件一般位于`$HOME/.gitconfig`
  - `git config --global user.name "user name"`：提交时的用户名，带空格时需要
  - `git config --global user.email xxx@xxx.com`：提交时使用的邮箱

    由于`github`等平台以邮箱为索引用户唯一依据，所以`user.email`是必须设置好的，否则上传到远程仓库的话贡献就不是自己的了

  - `git config --global core.editor nvim`：使用`git`进入编辑器界面时，会使用系统默认的编辑器，也可以通过`core.editor`配置修改
  - `git config --global core.pager bat`：使用`git log`或`git diff`时，输出将默认交给系统分页器如`less`，可配置修改
  - `git config --global url.<base>.insteadof <alias>`：对`url`中的`<base>`起别名`<alias>`
    - `git config --global url.git@github.com:.insteadof https://github.com/`：例如将`https`替换为`ssh`前缀，可以做到每次都直接使用`ssh`访问`github`的效果

      还可以替换为自己搭建的`github`镜像

  - `git config --global alias.<alias> <base>`：对`<base>`（如`checkout`等）起别名`<alias>`

- `git config --list`：查看所有配置
  - `--show-origin`：查看所有配置，同时显示来源
  - `--local/--global`：指定查看本地/全局
- `git config --get xxx`：查看配置
- `git config --get-regexp xxx`：模糊查询，使用简易的通配符匹配
- `git config --unset [xxx]`：重置某个设置

## 基本使用

### 创建仓库

- 创建本地仓库

  ```sh
  git init    # 新建空仓库
  git init [dir]  # 指定目录作为仓库
  git clone [href]  # 克隆别人的仓库, href即资源位置
  ```

  新建后，该文件夹内就会出现`.git`隐藏文件夹，可以进行提交等操作

  `href`在`github`中，你可以在一个项目的根目录下找到一个绿色的`Code`按钮，里面就有`url`

  `href`的一般格式：`https://github.com/用户名/仓库名.git`、`git@github.com:用户名/仓库名.git`

### 暂存与提交(`add`与`commit`)

- 暂存部分或全部文件的修改到**暂存区**：

  ```sh
  git add file1 file2 ...
  git add *.java
  git add .     # 暂存所有修改和新增
  git add . -- ":!target/"  # 暂存除了`target/`以外的
  git add -A    # 暂存所有修改、新增、删除
  git add -u [-- "xxx"]   # 暂存所有或部分文件的修改、删除，即不暂存未跟踪文件
  ```

- `git add -i`：打开交互式暂存
- `git add -p [xxx]`：对全部文件或部分文件打开选择式暂存修改(`hunk`)，会针对每一个`hunk`询问
  - y - stage this hunk
  - n - do not stage this hunk
  - q - quit; do not stage this hunk or any of the remaining ones
  - a - stage this hunk and all later hunks in the file
  - d - do not stage this hunk or any of the later hunks in the file
  - j - leave this hunk undecided, see next undecided hunk
  - J - leave this hunk undecided, see next hunk
  - g - select a hunk to go to
  - / - search for a hunk matching the given regex
  - e - manually edit the current hunk
  - ? - print help

- `git status`：检查暂存区内的内容，这些内容和`commit`时的那些注释内容是一致的
  - `-s`：简短查看，使用`M`、`A`、`D`、`??`等标志
- 将暂存区内容提交到仓库：

  ```sh
  git commit -m "explanation" # -m后为注释, 一般只适用于只有标题的提交
  git commit # 进入编辑器编辑提交信息
  ```

  一般来说，每次更新或修复一个功能，就需要提交一次，因此，上述命令是很常用的

- `git commit --amend`：追加提交，将暂存区提交到上一个提交，这种操作会产生新的哈希值
- `git commit --allow-empty`：空提交，通常用于触发`CI/CD`
- `git commit -p`：补丁式提交
- `git commit --no-edit`：不进入编辑器直接提交

### 分支管理(`branch`与`switch`)

- 分支在此前基本概念一节已经介绍过了，`Git`将本地分支存储在`.git/refs/heads/`下，远程分支存储在`.git/remtoes/`中
- `git branch`子命令用于管理分支
- 查看分支：
  - `-a`：查看所有分支，包括远程分支，远程分支指的是本地中跟踪了远程仓库的分支，仍然存储在本地
  - `-v`：查看所有本地分支同时显示最新提交
  - `-vv`：在`-v`的基础上额外显示最新提交上是否有上游的远程分支
  - `--merged`：查看已经合并的分支
  - `--no-merged`：查看未合并的分支
  - `-r`：查看所有远程分支
- 创建分支：
  - `git branch <branchname>`：基于当前分支创建新分支`<branchname>`
  - `git branch <newb> <commit/branch>`：基于指定提交/分支创建新分支
- 迁移分支：
  - `git branch -d <bname>`：删除某分支，无法删除所在分支
  - `git branch -D <bname>`：强制删除某分支
  - `git branch -m <newname>`：重命名当前分支
  - `git branch -m <oldname> <newname>`：重命名指定分支
  - `git branch -u <remotename>/<bname>`：设置当前分支的上游分支

    等价于`--set-upstream-to=<rname>/<bname>`

    也可以在创建分支的同时使用`--track`参数直接设置上游分支

- 在`git 2.23`之前，使用`checkout`进行切换分支、恢复文件，因此网上的教程多有使用`checkout`的，但此后`checkout`被分割成`switch`和`restore`

  `git switch`用于切换分支
  - `-c <newb> [<base>]`：切换时若分支不存在则创建
  - `-f`：默认若本分支存在未暂存/未提交的更改，则无法切换，`-f`表示丢弃本地修改（包括工作区和暂存区）
  - `-m`：携带所有未提交的修改带到下一个分支
  - `-t`：创建分支时指定跟踪远程分支
  - `-`：切换到上一个分支

### 恢复更改(`restore`与`reset`与`rm`与`reflog`)

- `git restore`专注于工作区、暂存区和本地仓库`HEAD`指向提交之间的文件恢复，替代了`checkout`与`reset`的部分功能
- `git restore <file>`：丢弃工作区的修改恢复到`HEAD`状态，暂存区仍保留

  PS：老版本中为`git checkout -- <file>`，`--`消除歧义，但`checkout`这种方式不会提供操作日志，较为危险
  - `--worktree`：就是上述命令所默认使用的参数
  - `--staged`：将暂存区的文件移出到工作区

    PS：等价于`git reset HEAD <file>`

  - `--patch`：补丁式恢复
  - `--source=<commit>`：指定恢复的源，默认是`HEAD`

- `git reset <commit> [<file>]`专注于不同提交之间的文件恢复，如果不跟文件则默认对所有文件执行，`reset`只能在本分支移动`HEAD`
  - `--mixed`：是默认模式，移动分支指针，将暂存区移动到工作区，不修改工作区
  - `--soft`：只移动分支指针
  - `--hard`：移动分支指针、清空工作区和暂存区
- `git rm <file>`用于快捷地删除工作区内容和删除误暂存的内容
  - 默认情况下，删除工作区的文件同时停止跟踪
  - 使用`--cached`参数只停止跟踪，保留在工作区而不删除
  - 使用`rm --cached`配合`commit --amend`可以删除被误提交的二进制文件等（工作区不变）：

    ```sh
    git rm --cached target/
    git commit --amend --no-edit
    ```

    但这种方式只适用于最新一次提交中误提交的情况，若历史提交中存在，则只能使用`filter-repo`

- `git reflog`用于查看操作日志

### 查看差异(`diff`)

- `diff`子命令用于精确地查看不同区之间的差异
- `diff`输出的结构：

  ````diff
  diff --git a/source/_posts/git.md b/source/_posts/git.md
  index b4a4d11..301e28f 100644
  --- a/source/_posts/git.md
  +++ b/source/_posts/git.md
  @@ -117,6 +117,8 @@ tags:
     - e - manually edit the current hunk
     - ? - print help

  +- `git status`：检查暂存区内的内容，这些内容和`commit`时的那些注释内容是一致的
  +  - `-s`：简短查看，使用`M`、`A`、`D`、`??`等标志
   - 将暂存区内容提交到仓库：

     ```sh
  ````

  第一行：`a/xxx`表示修改前的文件（即暂存区），`b/xxx`表示工作区文件

  第二行表示版本哈希值和文件权限

  第三、四行表示旧文件、新文件的路径

  **第五行**表示块头，`@@ -旧起始行号,旧行数 +新起始行号,新行数 @@`

  默认带有上下三行上下文，可以用`git diff -Ux`来显示`x`行

- `git diff`：查看工作区相对于暂存区的区别
- `git diff --staged`：查看暂存区相对于本地仓库之间的区别

  旧版本中为`git diff --cached`

- `git diff HEAD`：
- `git diff commitA commitB`，等价于`..`
- `git diff xxx...xxx`：三点运算符比较
- 一些参数：
  - `--stat`：显示每个文件的变更统计
  - `--shortstat`：显示每个文件的简短变更统计
  - `--name-only`：仅显示文件名称
  - `--name-status`：显示文件状态符号和文件名称
  - `--numstat`：显示变更行数统计

### 合并提交(`merge`与`rebase`与`cherry-pick`)

- `git merge`用于将两个分支合并，创建一个新的合并提交
  - `git merge xxx`：使`xxx`分支合并到当前分支
  - 快进合并：合并者是被合并者的直接祖先，则合并者直接移动到被合并者，不产生合并提交

    ```text
    [main] A---B---C
                    \
    [branch1]        D---E---F

    (main) git merge branch1

    [main] A---B---C---D---E---F
    [branch1]                  |
    ```

  - 三方合并：合并者本身有一定修改，则创建一个合并提交

    ```text
    [main] A---B---C
                \
    [branch1]    D---E---F

    (main) git merge branch1

    [main] A---B---C-------G
                \         /
    [branch1]    D---E---F
    ```

  - 默认情况下，会保留被合并者的所有提交记录
  - `--squash`：将被合并者的所有提交合并成一个提交，此时需要手动`commit`

- `git rebase`用于变基，即重新设置基准点而不是创建合并提交

  发起者会提取基准分支的额外提交（如下图的`C`）并以基准分支的最新提交作为新的基准点，然后在这个新的基准点的基础上应用本分支的历史提交（`DEF`），它们的提交哈希值会更改

  ```text
  [main] A---B---C
              \
  [branch1]    D---E---F

  (branch1) git rebase main

  [main] A---B---C
                  \
  [branch1]        D'---E'---F'
  ```

  注意`rebase`通常是在非主分支上执行的，和`merge`、`cherry-pick`不同，`rebase`通常用于整理这个分支的提交，使其看起来是线性的

  如果一个分支已经`push`上去了，就不要用`rebase`了，因为这会导致大量的更改，使得其它在这个分支上工作的协作者全都要额外处理，而且`rebase`产生冲突的概率是最大的

- `git cherry-pick xxx ...`用于复制提交，也不是创建合并提交，而是在源分支中自定义选择一些提交，复制到当前分支上，创建若干等量的新提交

  ```text
  [main] A---B---C
              \
  [branch1]    D---E---F

  (main) git cherry-pick branch1~..branch1~2

  [main] A---B---C---D'---E'
              \
  [branch1]    D---E---F
  ```

- 在使用上面三条命令时都有可能遇到合并冲突，冲突发现后使用任何命令都会失败并提醒
  - `git merge/rebase/cherry-pick --abort`：中止合并
  - `git merge/rebase/cherry-pick --continue`：解决冲突后继续合并
  - 冲突内容：会是下面的格式

    ```diff
    <<<<<<< HEAD
    ours
    =======
    theirs
    >>>>>>> branchname
    ```

    可以手动编辑文件直到没有冲突为止，也可以通过`checkout --ours`和`checkout --theirs`完全采纳我们的和它们的

    这里的`ours`始终是对于当前分支所说的，`theirs`始终是对于被合并分支来说的

    解决冲突永远修改的都是当前分支中发生冲突的提交，不会修改被合并分支

### 查看提交(`log`与`show`)

- `git log`用于查看提交历史
  - `--oneline`：单行查看，只显示`title`
  - `--graph`：显示分支结构
  - `-x`：显示最近`x`个提交
  - `-p`：显示每次提交的差异
  - `--stat`：显示统计
- `git show [commit/branch/tag]`用于查看提交详情

### 存储更改(`stash`与`worktree`)

- `git stash`用于暂时存储工作区+暂存区，从而能够切换分支而既不提交也不丢弃
  - `git stash`等价于`git stash push`，将当前`index`包括工作区更改存储起来
    - `-u`：这里不是`update`而是`untracked`，存储未跟踪文件
    - `-a`：包括所有文件，包括被`.gitignore`忽略的
  - `git stash list`：查看存储列表，每个存储项的格式：

    `stash@{x}: WIP on branch_name: commit_hash commit_title`

    `WIP`就是半成品的意思

  - `git stash pop [stash@{x}]`：恢复最近的存储，可以指定存储记录，同时将存储记录删除
  - `git stash apply [stash@{x}]`：同`pop`但不会删除存储记录
  - `git stash drop [stash@{x}]`：删除指定存储记录
  - `git stash clear`：删除所有记录
  - `git stash show [stash@{x}] [-p]`：查看存储内容，`-p`表示是否详细`diff`

- `git worktree`用于创建一个子目录，是另一种解决无法灵活切换分支的办法

  它将创建一个指定的新的目录，其中的`.git/`符号链接指向当前目录的`.git/`，则在新目录下的提交可以直接关联到原目录
  - `git worktree add <dir> <branch_name>`：新增，此后可以用`cd`来等效切换分支了
  - `lock`与`unlock`：加锁与解锁，用于防止意外删除
  - `git worktree remove <dir>`：删除
  - `git worktree list`：查看工作树

### 其它命令

- `git tag <tagname>`：标记本提交为一个自定义`tag`
- `git clean`：清理未跟踪的文件
- `git bisect`：用于二分查找引入`bug`的第一个提交
  - `git bisect start`：开始二分查找
  - `git bisect bad`：标记当前为坏提交
  - `git bisect good <tag>`：标记一个已知的好提交，即没有`bug`的，随后将自动跳转到中间的某个提交
  - `git bisect bad/good`：标记当前提交为坏/好，然后自动跳转到中间
  - `git bisect reset`：提前结束查找

## 远程仓库相关

### 管理远程仓库(`remote`)

- `git remote`子命令用于管理远程仓库
- 空本地仓库关联远程仓库

  ```sh
  git remote add [alias] [url] # 关联到某个远程仓库, alias是这个仓库的别名
  ```

- `git remote rename/remove/set-url/show`等
- 常见的工作流为：先`fork`源仓库，然后关联自己的这个远程仓库，此后`fetch`上游仓库后合并到本地仓库中

  ```sh
  git remote add origin [https://github.com/me/xxx.git]    # fork 下来的远程仓库通常命名为 origin
  git remote add upstream [https://github.com/other/xxx.git]    # 源仓库通常命名为 upstream (上游)
  git pull origin   # 直接 pull 下来
  # 本地完成功能后, 先合并上游的新更新再push到origin
  git fetch upstream
  git merge upstream/main
  git push origin
  ```

- 已有本地仓库关联远程仓库

  ```sh
  git remote add origin [https://github.com/me/xxx.git]
  git push origin   # 直接 push 即可
  ```

### 下载(`fetch`与`pull`与`push`)

- `git fetch remote_name [branch_name]`用于下载指定的远程仓库并存储到本地的远程分支中，默认下载全部分支
- `git fetch --prune`：清理已删除的远程分支引用
- `git pull`用于一键式地`fetch+merge`，可以使用`--rebase`表示`fetch+rebase`
- `git push remote_name localbr:remotebr`用于将本地提交推送
  - `git push rname abc`等价于`git push rname abc:abc`，即默认推送到同名分支

    这里是真正的推送到了远程仓库的分支里，而不是本地仓库的远程分支

  - `-u`：推送并设置上游跟踪

    设置上游分支后，此后只需要`git push`即可

  - `--delete xxx`：删除远程仓库的对应分支

    或者`git push origin :xxx`

  - `git push origin <tag>`：推送某个`tag`，使用`--tags`推送所有`tag`

### 安全事项

- 不要对已经`push`的提交进行修改，比如：
  - `push`了一个提交后`amend`
  - `push`了一个分支后尝试`rebase`，导致修改了这个分支上的所有提交
- 被`merge`后的分支应该删除，而不是继续在上面提交
- 从安全的角度上，`pull`缺少了审查步骤，应尽量使用分步骤地`fetch+merge`而不是`pull`

## `.gitignore`

- `.gitignore`中匹配的文件会被`git`忽略，它们甚至不会成为未跟踪文件
- `#`开头是注释
- `*`匹配任意字符（非目录分隔符）
- `?`匹配单个字符
- `**`匹配任意字符
- `[]`匹配字符类
- 以`/`结尾表示忽略目录

## `.gitattribute`

- `.gitattribute`是一系列预定义的处理规则，可以对不同文件进行不同的处理，包括设置处理工具、合并策略、换行符等
- 语法在这里不多赘述

## 轶闻

### `diff`的默认算法[`Myers`](https://link.springer.com/article/10.1007/BF01840446)

- ![myers](myers.png)
- 如图，`myers`通过层次遍历的方法，寻找`(0, 0)`到`(n, m)`的最短路径
- 横轴的每个点代表原文件的每一行，纵轴的每个点代表着新文件的每一行，因此`n`是原文件的总行数，`m`是新文件的总行数
- 允许的操作是向右、向下、向右下移动

  向右移动(`x->x+1`)意味着删除原文件的第`x+1`行，向下移动(`y->y+1`)意味着新增新文件的第`y+1`行，它们的消耗都为一

  向右下移动(`(x,y)->(x+1,y+1)`)意味着原文件的第`x+1`行和新文件的第`y+1`行匹配成功，花费为零但有条件(两文件的对应行一致)

- `diff`的目标就是找到人类习惯上的好的差异：差异尽可能紧凑、尽可能少，不变的行尽可能多

  而自然，最短路径将是右下操作最多的，刚好和好的差异相匹配

  寻找最短路径的过程就是宽度优先遍历并找到最先到达`(n, m)`的路径然后回溯，自然根据规则就能得到新增、删除的对比补丁了

  那么一个`hunk`自然就是连续的修改块了

### `git`集成插件

- `lazygit`是一个命令行工具，提供更好的`UI`界面和快捷键
- `neogit`是一个在`nvim`中的另一个集成插件，也是提供了更好的`UI`和快捷键
- `vimdiff`是一个能够更直观地查看`diff`以及修改冲突的工具

