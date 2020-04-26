---
title: Git 修改的存储、撤销与恢复
toc: true
tags:
    - git
categories:

---

![文件的状态变化周期](https://git-scm.com/book/en/v2/images/lifecycle.png)

<!-- more -->

理解 `reset` 和 `checkout` 命令请参考 [Git 工具 - 重置揭密](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)。


## <div id="_git_add">git add</div>
`git add <file>` 命令很简单，就是将文件添加到暂存区。

`<file>` 支持通配符，比如 `git add *.txt` 将所有格式为 txt 的文件添加到暂存区。

可以使用 `git add -A` 或 `git add .` 添加所有本地修改的文件（包括未跟踪的文件）。


## <div id="_git_checkout">git checkout</div>
`git checkout` 命令用于撤消对文件的修改，<font color="red"><b>该命令会删除文件在本地的任何修改，请谨慎使用该命令。</b></font>

```shell
$ git log --pretty=oneline
f48884c1ae2b1427e3d33d22345d66f30fe5f23c Add README file
dcf611e337c97ea23b230e841516cae7935fd1e4 add v3 file.txt
a0b947eb9049ad83e3f85a383e5223783fc43065 add v2 file.txt
c8232794435b45058018355dad81e3a289ed0192 add v1 file.txt

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   file.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	test.txt

$ git checkout .

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	test.txt
```

如上所示，`checkout` 命令只作用于已跟踪但尚未提交到暂存区的文件，未跟踪文件以及已暂存文件不会受到影响。


## <div id="_git_reset">git reset</div>

### <div id="_git_reset_to_unstage_file">取消暂存的文件</div>
使用 `git add .` 命令将本地修改全部添加到了暂存区 ，提交时发现某些文件不在本次提交范围内，可以使用 `git reset <pathspec>...` 命令将文件移出暂存区：
```shell
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
	modified:   file.txt

$ git reset README.md
Unstaged changes after reset:
M	README.md

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   file.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md
```

`reset` 命令后面参数为路径名时，不用担心误使用 `--hard` 模式选项会导致本地修改丢失。如果参数为路径名，同时指定模式名称，系统会提示错误：
```shell
$ git reset --hard test.md
fatal: Cannot do hard reset with paths.

$ git reset --soft test.md
fatal: Cannot do soft reset with paths.
```


## <div id="_git_stash_command">git stash</div>
在开发中经常遇到这种情况，在 `dev` 分支上开发新功能时，有新的 bug 需要马上解决，此时新功能尚未开发完，不想增加一个脏提交，这时可以使用 `git stash` 命令。`git stash` 用于保存和恢复工作状态。

### <div id="_git_stash_list">查看存储列表</div>
使用 `git stash list` 查看已存储状态的列表：
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md
```

### <div id="_git_stash_show">查看指定状态的 diff</div>
使用 <code>git stash show [-p|--patch] [<stash\>]</code> 查看指定 stash 的 diff：
```shell
$ git stash show
 test2.md | 1 +
 1 file changed, 1 insertion(+)

 $ git stash show -p stash@{1}
 diff --git a/test1.md b/test1.md
 new file mode 100644
 index 0000000..e4fc91c
 --- /dev/null
 +++ b/test1.md
 @@ -0,0 +1 @@
 +"This is test1.md file"

 $ git stash show --patch stash@{2}
diff --git a/test.md b/test.md
new file mode 100644
index 0000000..4a32ea6
--- /dev/null
+++ b/test.md
@@ -0,0 +1 @@
+"This is test.md file"
```

`git stash show [<stash>]` 显示改动的信息，即哪些文件发生了改变，有多行产生了改动。

`git stash show -p [<stash>]` 显示改动的具体内容。

### <div id="_git_stash_save">保存状态</div>
保存当前工作目录中的状态：`git stash save [-u|-a] "message"`

`git stash` 保存所有 **已跟踪** 文件的修改，但不保存 **未跟踪** 的文件（git 文件状态请参考[记录每次更新到仓库](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%AE%B0%E5%BD%95%E6%AF%8F%E6%AC%A1%E6%9B%B4%E6%96%B0%E5%88%B0%E4%BB%93%E5%BA%93)）。

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

$ git stash
Saved working directory and index state WIP on master: bb764d6 Add locale youtube download shell

$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

nothing added to commit but untracked files present (use "git add" to track)

$ git stash
No local changes to save
```

如上所示，`git stash` 不会存储尚未跟踪的文件 test.md，保存未追踪的文件系统会提示 “No local changes to save”。

使用 `git stash -u|--include-untracked` 在存储时包含未跟踪的文件：
```shell
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

$ git stash save -u "stash files include untracked files"
Saved working directory and index state On master: stash files include untracked files

$ git status
On branch master
nothing to commit, working tree clean
```

无论是 `git stash` 或 `git stash -u` 都不包含被忽略的文件。如果想要包含被忽略的文件，则需要使用 `git stash -a|--all` 命令，该命令会将所有已跟踪文件、未跟踪文件以及被忽略的文件存储到状态中：
```shell
$ cat .gitignore
log/

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

$ ls -l log\
total 1
-rw-r--r-- 1 neo1949 197609 23 4月  12 01:04 log.txt

$ git stash save -a "stash files include untracked and ignored files"
Saved working directory and index state On master: stash files include untracked and ignored files

$ git status
On branch master
nothing to commit, working tree clean
```

实际开发时，**禁止** 直接使用 `git stash` 命令，而是使用 `git stash save [-u|-a] "message"` 在保存状态时添加修改说明。禁止理由如下：有时保存了工作状态，但相关功能暂时不需要了，添加修改说明可以方便日后快速预览所有状态，不用再比对 diff 反推当时为何添加某个修改。相信我，如果你时隔几个月再去查看以前保存的某个状态，一定会赞同此说法。

### <div id="_git_stash_restore">恢复状态</div>

#### <div id="_git_stash_pop">恢复并删除状态</div>
恢复状态的同时从存储列表中删除该状态： `git stash pop [--index] [<stash>]`
```shell
$ git stash list
stash@{0}: On master: save test2.md
stash@{1}: On master: save test1.md
stash@{2}: On master: save test.md

$ git stash pop
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test2.md

Dropped refs/stash@{0} (bf0fba50dd944507668419b6502acceb935eb262)

$ git stash list
stash@{0}: On master: add test1.md
stash@{1}: On master: add test.md

$ git stash pop stash@{1}
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test.md
        new file:   test2.md

Dropped stash@{1} (ecee645f4fff9e493fa56dd6df9eccd579e90114)

$ git stash list
stash@{0}: On master: add test1.md
```

#### <div id="_git_stash_apply">只恢复状态</div>
恢复状态时，不会从存储列表中删除该状态： `git stash apply [--index] [<stash>]`
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash apply
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test2.md

$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash apply stash@{1}
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md
```

### <div id="_git_stash_drop">删除状态</div>
删除指定状态： `git stash drop [<stash>]`
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash drop
Dropped refs/stash@{0} (579fcf40727d31a15c76065ff465ef66d8ba4805)

$ git stash list
stash@{0}: On master: add test1.md
stash@{1}: On master: add test.md

$ git stash drop stash@{1}
Dropped stash@{1} (561d3c994a0d1e4415483776fcaa0d5db42929ff)

$ git stash list
stash@{0}: On master: add test1.md
```

删除所有状态： <code>git stash clear</code>
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash clear

$ git stash list
```

**注意：** 删除状态时，修改的内容也会被删除。


## <div id="_resotre_changes">恢复修改</div>

### <div id="_restore_committed_changes">恢复本地已提交的修改</div>
本地修改已经 `commit`，然后某个时刻在分支中使用了 `git reset --hard <commit>` 命令重置了索引和目录树，`reset` 后发现修改还没有 `push` 到服务器，可以使用以下方式恢复修改。

示例：本地提交后重置了当前分支的 `HEAD`
```shell
$ git log --abbrev-commit --pretty=oneline -3
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
2617b13 Merge branch 'test'
0c61754 Merge branch 'tmp'

$ vim git.md

$ git add git.md

$ git commit -m "Add git command summary"
[master ef88a9c] Add git command summary
 1 file changed, 1 insertion(+)
 create mode 100644 git.md

$ git log --abbrev-commit --pretty=oneline -3
ef88a9c Add git command summary
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
2617b13 Merge branch 'test'

// commit 之后未 push 到服务器执行了 reset 命令
$ git reset --hard 2617b13
HEAD is now at 2617b13 Merge branch 'test'

$ git log --abbrev-commit --pretty=oneline -3
2617b13 Merge branch 'test'
0c61754 Merge branch 'tmp'
b1de099 Merge branch 'dev'
```

现在需要将 `ef88a9c` 对应的提交推送到服务器 ，那么首先需要恢复该修改，有如下方式。

#### <div id="_use_sha1_to_restore_uncommitted_changes">利用 SHA-1 校验和恢复修改</div>
```shell
$ git reset --hard ef88a9c
HEAD is now at ed74a3e Add git command summary

$ git checkout ef88a9c
Note: checking out 'ef88a9c'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at ef88a9c... Add git command summary
```

#### <div id="_use_reflog_to_restore_uncommitted_changes">利用 `reflog` 命令恢复修改</div>
```shell
$ git reflog
2617b13 HEAD@{0}: reset: moving to 2617b13
ef88a9c HEAD@{1}: commit: Add git command summary
13f83a3 HEAD@{2}: clone: from https://github.com/neo1949/GitTest.git

$ git reset --hard HEAD@{1}
HEAD is now at ed74a3e Add git command summary

$ git checkout HEAD@{1}
Note: checking out 'HEAD@{1}'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at ef88a9c... Add git command summary
```

### <div id="_changes_unable_to_restore"><font color="red">无法恢复修改的情形</font></div>
如果使用 `git checkout <pathspec>` 命令撤销了未提交到暂存区的本地修改，`git` 没有可用命令恢复修改。这种时候可以使用软件的 `Ctrl + Z` 回退功能：立即找到被撤销修改的文件执行回退操作，如果软件的回退功能也无法找回修改，那么只能 “恭喜” 你了！

使用 `git reset --hard <commit>` 重置分支 `HEAD` 时，如果本地修改尚未提交，那么本地已跟踪文件的修改同样会丢失，并且无法恢复：
```shell
$ git st
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   test.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	Test.java

no changes added to commit (use "git add" and/or "git commit -a")

$ git log --abbrev-commit --pretty=oneline -3
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
2617b13 Merge branch 'test'
0c61754 Merge branch 'tmp'

$ git reset --hard 0c61754
HEAD is now at 0c61754 Merge branch 'tmp'

$ git status
On branch master
Your branch is behind 'origin/master' by 3 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	Test.java

nothing added to commit but untracked files present (use "git add" to track)
```

如果文件处于 `Untracked` 状态，那么文件不会丢失，这可能是使用 `reset` 命令导致修改丢失唯一不那么难受的事情了。

<font color="red">因此，谨慎以下两种情形，因为本地修改可能无法恢复：</font>
1. 使用 `git checkout <pathspec>` 撤销本地已跟踪文件的修改
2. 本地修改未提交时使用 `git reset --hard <commit>` 命令
