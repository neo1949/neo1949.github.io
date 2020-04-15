---
title: Git 存储、撤消、恢复
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
本地修改了的多个文件需要作为两次独立的修改提交，习惯性使用 `git add .` 暂存了所有的修改，可以使用 `git reset <file>` 命令将文件移出暂存区：
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

`git reset <file>` 不会删除文件在本地的修改，可以放心使用。

```shell
$ git log --abbrev-commit --pretty=oneline
dcf611e add v3 file.txt
a0b947e add v2 file.txt
c823279 add v1 file.txt

$ git reset dcf611e

$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md

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
