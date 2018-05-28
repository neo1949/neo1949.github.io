---
title: Git常用命令总结
date: 2018-03-19 14:56:37
tags:
    - Git
categories:
---

首先建议大家阅读 [官方文档](https://git-scm.com/book/zh/v2/)，里面详细的介绍了 Git 的前世今生。官方文档基本提供了你需要的任何跟 Git 相关的东西，并且文档非常友好地提供了中文版。

<!-- more -->

既然官方文档有中文版了，为什么还要写这篇文章呢？我的目的当然不是为了重复造轮子，主要是用来记录一些平时开发常用的命令，养成良好的总结和记录习惯对我们很有帮助，希望大家也能养成这个习惯。

## 安装和配置 Git
安装 Git 请参照 [安装 Git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)，配置 Git 请参照 [初次运行 Git 前的配置](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C-Git-%E5%89%8D%E7%9A%84%E9%85%8D%E7%BD%AE)。

### 查看配置信息
```
git config --list
```

### 配置单个项目的用户名和邮箱
有时我们想在当前项目中使用特定的用户名和邮箱，只要去掉 <code>-\-global</code> 选项或者使用 <code>-\-local</code> 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里：
```
git config [--local] user.name "local.name"
git config [--local] user.email "local.email"
```
查看配置信息可以看到末尾新增了刚才设置的用户名和邮箱。

想要撤销单个项目配置的用户名和邮箱也很简单，只需要使用下面的命令即可：
```
git config --unset user.name
git config --unset user.email
```
再次查看配置信息，可以看到本地配置的用户名和邮箱已经被删除了，提交代码时会默认使用全局配置的用户名和邮箱。

## 获取帮助的三种方式
可以通过以下三种方法可以找到 Git 命令的使用手册：
```
git help <verb>
git <verb> --help
man git-<verb>
```

例如查看 config 命令的手册，执行：
```
git help config
```

## Git 的三种状态
在正式使用 Git 之前，需要了解 Git 中几个非常重要的概念。

[Git 基础](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-Git-%E5%9F%BA%E7%A1%80) 写到 “Git 有三种状态，你的文件可能处于其中之一：已提交 <code>committed</code>、已修改 <code>modified</code> 和已暂存<code>staged</code>”。实际上这段描述翻译地不是特别准确，英文原文是 "Git has three main states that your files can reside in: *committed*, *modified*, and *staged*"。原文实际上说的是 “你的文件在 Git 主要有三种状态：已提交 <code>committed</code>、已修改 <code>modified</code> 和已暂存 <code>staged</code>”，意思是主要有三种状态，而不是只有三种状态的意思。因为这可能给后面 Git 文件状态变化的理解造成误导，所以特地指出。

<code>committed</code> 表示数据已经安全的保存在本地数据库中。<code>modified</code> 表示修改了文件，但还没保存到数据库中。<code>staged</code> 表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。

![工作目录、暂存区域以及 Git 仓库](https://git-scm.com/book/en/v2/images/areas.png)

- Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。
- 工作目录是对项目的某个版本独立提取出来的内容。这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
- 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。有时候也被称作‘索引’，不过一般说法还是叫暂存区域。

基本的 Git 工作流程如下：
1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

如果 Git 目录中保存着特定的版本文件，就属于已提交状态。如果作了修改并已放入暂存区域，就属于已暂存状态。如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

工作目录下的每一个文件都不外乎两种状态：已跟踪或未跟踪。已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。下图是 Git 中文件的状态变化周期：

![文件的状态变化周期](https://git-scm.com/book/en/v2/images/lifecycle.png)

在使用 Git 命令时，需要牢记文件状态以及工作区域的概念。

## 获取 Git 仓库
### 创建一个新的仓库
```
git init
```

### <sapn id="clone_an_existing_repository">克隆现有的仓库</span>
```
git clone url [local-repo-name]
```
不指定本地仓库 <code>local-repo-name</code> 名称时，系统会默认创建与远程仓库名称相同的文件夹并将仓库拷贝到文件夹中。

##  记录每次更新到仓库
1.查看当前文件的状态
```
git status
```

2.跟踪新文件
```
git add filename
```
也可以使用 <code>git commit -A</code> 命令将所有文件添加到暂存区域。

3.查看修改
```
git diff [--cached] [filename]
```
<code>git diff</code> 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。如果不指定文件名称，默认会比较本地所有已跟踪但未暂存的修改。

若要查看已暂存的将要添加到下次提交里的内容，可以用 <code>git diff -\-cached</code> 命令。（Git 1.6.1 及更高版本还允许使用 <code>git diff -\-staged</code>，效果相同但更好记些。）

4.移除文件
```
git rm filename
```
要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。可以用 <code>git rm</code> 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

**5.将文件从暂存区移除**
```
git rm --cached filename
```
已添加到暂存区的文件会被移除，但是本地的修改不会被删除，仍然保留工作在工作目录中。

6.移动文件
```
git mv file_from file_to
```
如果我们想要重命名文件或者将文件移动到另一个文件夹下面，可以使用此种方式。

7.提交更新到本地仓库
```
git commit -m "commit message"
```

## Git 提交记录 - git log
```
git log
```
按提交时间列出所有的更新，列出每个提交的 <code>SHA-1</code> 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

1.显示指定文件所有的更新
```
git log filename
```

2.显示最近n次的提交记录
```
git log -n
```

3.显示每次提交的内容差异
```
git log -p
```

以上参数可以混合使用。例如，查看 <code>README</code> 文件最近两次提交的内容差异：
```
git log -p -2 README.md
```

4.查看每次提交的简略统计信息
```
git log --stat
```

<code>-\-stat</code> 选项在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是添加了。在每次提交的最后还有一个总结。

5.使用 <code>-\-pretty=oneline</code> 将每个提交放在一行显示，查看的提交数很大时非常有用
```
git log --pretty=oneline
```

6.显示指定作者相关的提交
```
git log --author="author_name"
```

7.显示包含指定关键字的提交
```
git log --grep="keyword"
```

Git 强大的地方不仅在于提供了丰富的命令，更重要的是可以组合使用这些命令找到我们想要的结果。比如，我想查看作者 neo1949 提交中包含关键字 test 最近的5条记录，可以使用如下的方式：
```
git log --author=neo1949 --grep=test -5
```
这里需要注意的是：因为作者名不包含空格，所以可以不使用<code>""</code>引用，关键字同理。如果你的用户名包含关键字或者关键字是短语，那么必现要使用<code>""</code>。

关于 <code>git log</code> 更详细的用法请参考 [查看提交历史](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2) 章节。

## 撤消操作
1.覆盖提交
```
git commit --amend
```
有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。此时，可以先将未添加的文件添加到暂存区后（更新提交信息跳过即可），再使用 <code>-\-amend</code> 选项提交命令尝试重新提交。

2.取消暂存的文件但保留本地修改
```
git reset HEAD filename
```
与 <code>git rm -\-cached filename</code> 命令作用相同，都是取消暂存的文件，但是本地的修改仍在，工作内容不会丢失。

3.撤消对文件的修改
```
git checkout filname
```
撤消本地文件的所有修改，本地所有修改的工作内容都会被删除。需要注意的是，只有文件处于已跟踪但未暂存的状态，此命令才会生效。如果是新增的文件或者已添加到暂存区的文件，使用这个命令不会删除本地的修改。


## 远程仓库的使用
### 查看远程仓库
如果想查看你已经配置的远程仓库服务器，可以运行 <code>git remote</code> 命令。它会列出你指定的每一个远程服务器的简写。如果你已经克隆了自己的仓库，那么至少应该能看到 <code>origin</code> - 这是 Git 给你克隆的仓库服务器的默认名字：
```
$ git remote
origin
```

使用 <code>-v</code> 参数显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL：
```
$ git remote -v
origin  git@github.com:neo1949/GitTest.git (fetch)
origin  git@github.com:neo1949/GitTest.git (push)
```

### <span id="add_remote_git_repository">添加远程仓库</span>
使用下面的命令添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写：
```
git remote add <shortname> <url>
```
现在你可以在命令行中使用字符串 <code>shortname</code> 来代替整个 URL。

### 从远程仓库中拉取与抓取
1.从远程仓库中获得数据
```
git fetch
```

如果你使用 <code>clone</code> 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。所以，<code>git fetch origin</code> 会抓取克隆（或上一次抓取）后新推送的所有工作。必须注意 <code>git fetch</code> 命令会将数据拉取到你的本地仓库，但它并不会自动合并或修改你当前的工作。当准备好时你必须手动将其合并入你的工作。

2.推送到远程分支
```
git push [remote-name] [branch-name]
```

当你想要将 <code>master</code> 分支推送到 <code>origin</code> 服务器时（再次说明，克隆时通常会自动帮你设置好那两个名字），那么运行这个命令就可以将你所做的备份到服务器：
```
$ git push origin master
```

3.查看远程仓库
```
git remote show [remote-name]
```

例如：查看远程仓库 <code>origin</code> 的详细信息
```
$ git remote show origin
* remote origin
  Fetch URL: git@github.com:neo1949/GitTest.git
  Push  URL: git@github.com:neo1949/GitTest.git
  HEAD branch: master
  Remote branches:
    master tracked
    tmp    tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```
这些信息非常有用，它会列出远程仓库的 URL 与跟踪分支的信息。它会告诉你正处于哪个分支。比如本例中可以看到你当前处于 <code>master</code> 分支，如果你运行 <code>git pull</code> 命令，Git 就会自动抓取所有的远程引用，然后将远程 <code>master</code> 分支合并到本地 <code>master</code> 分支。它也会列出拉取到的所有远程引用。

这个命令还会列出当你在特定的分支上执行 <code>git push</code> 命令式会自动地推送到哪一个远程分支。同时还会列出哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了，还有当你执行 <code>git pull</code> 时哪些分支会自动合并。

4.<span id="rename_remote_repository">重命名远程仓库</span>
```
git remote rename old-remote-name new-remote-name
```
值得注意的是这同样也会修改你的远程分支名字。那些过去引用 <code>old-remote-name/master</code> 的现在会引用 <code>new-remote-name/master</code>。

5.移除远程仓库
```
git remote rm remote-name
```
如果因为一些原因想要移除一个远程仓库：你已经从服务器上搬走了或不再想使用某一个特定的镜像了，又或者某一个贡献者不再贡献了，可以使用上面的命令删除某个远程仓库。

## Git 标签 - git tag
Git 可以给历史中的某一个提交打上标签，通常用来标记发布结点。

### 列出标签
```
git tag
```

### 创建标签
```
git tag -a tag-name -m "tag message"
```

例如：
```
$ git tag -a v0.2 -m "Add tag v0.2 for test"
$ git tag
v0.1
v0.2
```

### 查看标签信息
```
git show tag-name
```

### 后期打标签
如果我们在某次提交忘记给项目打标签，可以之后补上标签。要在那个提交上打标签，只需要在命令的末尾指定提交的校验和（或部分校验和）:
```
git tag -a tag-name commit-hash-id
```

### 共享标签
默认情况下，<code>git push</code> 命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显式地推送标签到共享服务器上。这个过程就像共享远程分支一样:
```
git push origin tag-name
```

如果想要一次性推送很多标签，也可以使用带有 <code>-\-tags</code> 选项，这将会所有不在远程仓库服务器上的标签全部推送到远程仓库中：
```
git push origin --tags
```

这样其它人拉取仓库的时候也能看到这些标签了。

### 检出标签
在 Git 中你并不能真的检出一个标签，因为它们并不能像分支一样来回移动。如果你想要工作目录与仓库中特定的标签版本完全一样，可以在特定的标签上创建一个新分支并切换到该分支：
```
git checkout -b [branch-name] [tag-name]
```

通过该命令可以实现下载某个仓库指定版本的代码，例如：
```
$ git clone git@github.com:neo1949/GitTest.git GitTest2
$ cd GitTest2
$ git tag
v0.1
v0.2
v0.3
$ git checkout v0.2
```
有时候我们会需要一个特定版本的程序，可以通过上面的方法先下载指定版本的代码，然后编译程序即可。

## Git 别名
强烈建议不要一开始就使用别名，一定要在熟练使用 Git 之后再使用别名，具体请自行参考 [Git 别名](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-Git-%E5%88%AB%E5%90%8D)。

## Git 分支
### 分支简介
想要真正理解 Git 处理分支的方式，首先需要了解 Git 是如何保存数据的。

Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。

在进行提交操作时，Git 会保存一个提交对象（commit object）。知道了 Git 保存数据的方式，我们可以很自然的想到——该提交对象会包含一个指向暂存内容快照的指针。但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象：

![commit object](http://p5ia12npj.bkt.clouddn.com/commit_object.png)

当使用 <code>git commit</code> 进行提交操作时，Git 会先计算每一个子目录的校验和，然后在 Git 仓库中这些校验和保存为树对象。随后，Git 便会创建一个提交对象，它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。如此一来，Git 就可以在需要的时候重现此次保存的快照。

### 创建分支
Git 在创建分支时，只是创建了一个可以移动的新的指针：
```
git branch new-branch
```

Git 有一个名为 <code>HEAD</code> 的特殊指针，指向当前所在的本地分支。

使用 <code>git log -\-decorate</code> 可以查看各个分支当前所指的对象：
```
$ git log --decorate --oneline
aed99ae (HEAD, tag: v0.3, origin/master, master) Summary: add tag.md for test
67aa248 (tag: v0.2) Create TestFetch.md
4ee2795 Summary: remove Chapter_02_branch.md
a7e5abc Summary: update Chapter_02_branch.md
80d1cf2 Summary: add Chapter_02_branch.md
e07e4ca (origin/tmp, origin/dev, tmp, test, dev) Merge branch 'master' of github.com:neo1949/GitTest
2d9c922 Summary: update Chapter_01_init.md
a5d6af0 Summary: update Chapter_01_init.md
bb3fb11 Merge branch 'master' of github.com:neo1949/GitTest
b9c24d5 Summary: update Chapter_01_init.md
c11ceb1 Summary: update Chapter_01_init.md
c349273 Merge branch 'master' of github.com:neo1949/GitTest
0b90633 Summary: create a git repo
75b684b (tag: v0.1) Update README.md
0adb202 Initial commit
```
可以看到 <code>master</code> 分支指向 <code>aed99ae</code> 开头的提交对象，而 <code>tmp</code> 和 <code>dev</code> 分支则指向 <code>e07e4ca</code> 开头的提交对象。

### 查看分支
1.查看所有本地分支
```
git branch
```
<code>*</code> 表示当前所在的分支（也就是 <code>HEAD</code> 指针指向的分支）。

2.查看所有分支
```
git branch -a
```

3.查看所有远程分支
```
git branch -r
```

4.查看每个分支的最后一次提交
```
git branch -v
```

5.查看已经合并到当前分支的分支
```
git branch --merged
```

6.查看未合并到当前分支的分支
```
git branch --no-merged
```

### 切换分支
1.切换到指定分支：
```
git checkout branch-name
```

2.创建分支并切换到新的分支：
```
git checkout -b new-branch
```

创建一个新分支后并切换过去进行了一些工作，随后又切换回 master 分支进行了另外一些工作，提交到仓库之后，这个项目的提交历史就产生了分叉，可以用下面的方式查看项目分叉历史：
```
$ git log --decorate --oneline --graph --all
* aed99ae (HEAD, tag: v0.3, origin/master, master) Summary: add tag.md for test
* 67aa248 (tag: v0.2) Create TestFetch.md
* 4ee2795 Summary: remove Chapter_02_branch.md
* a7e5abc Summary: update Chapter_02_branch.md
* 80d1cf2 Summary: add Chapter_02_branch.md
*   e07e4ca (origin/tmp, origin/dev, tmp, test, dev) Merge branch 'master' of github.com:neo1949/GitTest
|\  
| * a5d6af0 Summary: update Chapter_01_init.md
* | 2d9c922 Summary: update Chapter_01_init.md
|/  
*   bb3fb11 Merge branch 'master' of github.com:neo1949/GitTest
|\  
| * c11ceb1 Summary: update Chapter_01_init.md
* | b9c24d5 Summary: update Chapter_01_init.md
|/  
*   c349273 Merge branch 'master' of github.com:neo1949/GitTest
|\  
| * 75b684b (tag: v0.1) Update README.md
* | 0b90633 Summary: create a git repo
|/  
* 0adb202 Initial commit
```

### <span id="merge_branches">合并分支</span>
分支的合并请仔细阅读 [分支的创建与合并](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6) 章节中“分支的合并”部分，主要记住合并分支的命令是 <code>git merge branch-name</code>。关于如何解决合并时的冲突，文档里面提供详细了的介绍，请仔细阅读。

下面我们将在本地模拟合并分支冲突的情形：

1.首先在主分支 <code>master</code> 创建一个空文件 "merge_conflict_test.md"，然后将其提交到远程仓库 <code>master</code> 分支上：
```
$ git add merge_conflict_test.md
$ git commit -m "Add merge_conflict_test.md file for test"
$ git push origin master
```

2.在本地创建一个 <code>merge_demo</code> 并切换到该分支：
```
git checkout -b merge_demo
```

3.打开 "merge_conflict_test.md" 文件，在第一行添加一句话：
> This line was added on merge_demo branch.

4.将文件提交到仓库后切换回主分支：
```
$ git add merge_conflict_test.md
$ git commit -m "Add a line in merge_conflict_test.md on merge_demo branch"
$ git checkout master
```
切回到主分支后查看 "merge_conflict_test.md" 文件，可以看到我们刚才添加的那行内容不见了。这是必然的，因为我们的修改是在 <code>merge_demo</code> 分支上进行的。

5.接下来再次在 "merge_conflict_test.md" 文件的第一行添加一句话：
> This line was added on master branch.

按照上面同样的方式将文件提交到仓库中：
```
$ git add merge_conflict_test.md
$ git commit -m "Add a line in merge_conflict_test.md on master branch"
```

6.下面将 <code>merge_demo</code> 合并到 <code>master</code> 分支中：
```
$ git merge merge_demo
Auto-merging merge_conflict_test.md
CONFLICT (content): Merge conflict in merge_conflict_test.md
Automatic merge failed; fix conflicts and then commit the result.
```

可以看到 Git 提示我们合并分支发生了冲突，需要我们解决冲突后提交结果。

7.查看此时文件 "merge_conflict_test.md" 的内容如下：
```
$ cat merge_conflict_test.md
<<<<<<< HEAD
This line was added on master branch.
=======
This line was added on merge_demo branch.
>>>>>>> merge_demo
```

这表示 <code>HEAD</code> 所指示的版本（也就是 <code>master</code> 分支所在的位置）在这个区段的上半部分（<code>=======</code> 的上半部分），而 <code>merge_demo</code> 分支所指示的版本在 <code>=======</code> 的下半部分。根据实际情况来确定需要删除和保留哪些内容。

8.解决冲突后再次将文件提交到仓库：
```
$ cat merge_conflict_test.md
This line was added on master branch.
This line was added on merge_demo branch.
This line was added to show how fixing conflict.
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   merge_conflict_test.md

no changes added to commit (use "git add" and/or "git commit -a")
$ git add merge_conflict_test.md
$ git commit
```

使用 <code>git commit</code> 提交时可以看到系统自动添加了哪个文件发生了合并冲突的提交说明：
```
Merge branch 'merge_demo'

# Conflicts:
#	merge_conflict_test.md
# ...
```

通过这个例子可以看到，如果在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们从而导致合并冲突。

现在我们应该已经学会如何解决合并冲突的问题了！

### 删除分支
1.<span id="delete_local_branch">删除本地分支</span>
```
git branch -d branch-name
```

2.强制删除本地分支（忽略冲突）
```
git branch -D branch-name
```

3.<span id="delete_remote_branch">删除远程仓库的分支</span>
```
git push origin --delete branch-name
```

官方文档对这个命令有一段说明：基本上这个命令做的只是从服务器上移除这个指针。Git 服务器通常会保留数据一段时间直到垃圾回收运行，所以如果不小心删除掉了，通常是很容易恢复的。

网上还介绍了一种删除远程分支的方法：
```
git push origin :branch-name
```
这种方式也可以删除远程仓库里的分支，但是是否会保留数据未知，所以如果误删除了某个分支，可能无法恢复数据，为了安全起见，不推荐使用此种方式删除远程分支。

删除远程仓库分支之后，使用 <code>git branch -a</code> 查看分支信息，发现刚才删除的远程分支依然能看到，可以使用 <code>git remote prune origin</code> 命令后再查看分支信息，不会再显示已删除的远程分支。

下面完整地演示如何将一个 <code>dev</code> 分支添加到远程仓库后再删除的流程：
```
$ git branch -a
* master
  remotes/origin/master
  remotes/origin/tmp
$ git branch dev
$ git push origin dev
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:neo1949/GitTest.git
 * [new branch]      dev -> dev
$ git branch -a
  dev
* master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/tmp
$ git push origin --delete dev
To git@github.com:neo1949/GitTest.git
 - [deleted]         dev
$ git branch -a
  dev
* master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/tmp
$ git remote prune origin
$ git branch -a
  dev
* master
  remotes/origin/master
  remotes/origin/tmp
```

### 远程分支
远程分支的概念请先阅读 [远程分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)。以下为个人摘录总结，可能存在理解错误。

远程引用是对远程仓库的引用（指针），包括分支、标签等等。可以通过 <code>git ls-remote (remote)</code> 来显式地获得远程引用的完整列表，或者通过之前介绍过的  <code>git remote show (remote)</code> 命令获得远程分支的更多信息。通常更常见的做法是利用**远程跟踪分支**。

远程跟踪分支是远程分支状态的引用，以 <code>(remote)/(branch)</code> 形式命名。你不能移动这些本地引用，当你做任何网络通信操作时，它们会自动移动。远程跟踪分支像是你上次连接到远程仓库时，那些分支所处状态的书签。使用 <code>git clone</code> 命令克隆仓库时，系统会自动将其命名为 <code>origin</code>，拉取它的所有数据，创建一个指它的 <code>master</code> 分支的指针，并且在本地将其命名为 <code>origin/master</code>。同时，Git 会在本地创建一个与 <code>origin</code> 的 <code>master</code> 指向同一个地方的 <code>master</code> 分支，这样可以方便你开展工作。

需要再次明确一点：<code>origin</code> 并无特殊含义。远程仓库名字 <code>origin</code> 与分支名字 <code>master</code> 一样，在 Git 中并没有任何特别的含义。<code>master</code> 是运行 <code>git init</code> 时默认的起始分支名字，原因仅仅是它的广泛使用。<code>origin</code> 是运行 <code>git clone</code> 时默认的远程仓库名字，如果你运行 <code>git clone -o booyah</code>，那么你默认的远程跟踪分支名字将会是 <code>booyah/master</code>。

演示：
```
$ git clone -o work git@github.com:neo1949/GitTest.git GitTest2
$ cd GitTest2
$ git branch -a
* master
  remotes/work/HEAD -> work/master
  remotes/work/master
  remotes/work/tmp
```

可以看到 <code>git clone -o remote-name</code> 实际上相当于先执行了 [克隆仓库](#clone_an_existing_repository) 的命令，然后又执行了 [重命名仓库](#rename_remote_repository) 的操作：
```
$ git clone git@github.com:neo1949/GitTest.git GitTest2
$ cd GitTest2
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/tmp
$ git remote rename origin work
$ git branch -a
* master
  remotes/work/HEAD -> work/master
  remotes/work/master
  remotes/work/tmp
```

所以，完整的克隆仓库命令如下：
```
git clone [-o shortname] url [local-repo-name]
```
- shortname ：使用 <code>shortname</code> 作为远程仓库名，不指定该参数时使用 <code>origin</code> 作为远程仓库名
- local-repo-name ：使用 <code>local-repo-name</code> 作为工作目录名称，不指定该参数时使用远程仓库名创建工作目录

#### 将本地分支推送远程仓库
```
git push remote-name branch-name[:remote-branch-name]
```

想要把一个分支推送到远程仓库上，必须要有这个仓库的写入权限。使用 <code>git push remote-name branch-name</code> 将本地分支推送到远程仓库中，系统会在远程仓库中创建一个名为 <code>branch-name</code> 的分支。如果我们并不想让远程仓库上的分支叫做<code>branch-name</code>，而是想使用一个新的分支名，可以使用 <code>git push remote-name branch-name:remote-branch-name</code> 的方式将本地 <code>branch-name</code> 分支推送到远程仓库上的 <code>remote-branch-name</code> 分支。

示例：
```
$ git branch local-dev
$ git push origin local-dev
$ git branch -a
local-dev
* master
remotes/origin/HEAD -> origin/master
remotes/origin/dev
remotes/origin/local-dev
remotes/origin/master
remotes/origin/team
remotes/origin/test
remotes/origin/tmp
```
可以看到远程仓库中多了一个 <code>local-dev</code> 分支。

```
$ git branch local-cache
$ git push origin local-cache:origin-cache
$ git branch -a
local-cache
local-dev
* master
remotes/origin/HEAD -> origin/master
remotes/origin/dev
remotes/origin/local-dev
remotes/origin/master
remotes/origin/remote-cache
remotes/origin/team
remotes/origin/test
remotes/origin/tmp
```
可以看到远程仓库多了一个 <code>remote-cache</code> 分支。

#### 删除远程分支
[删除远程分支](#delete_remote_branch) 和 [删除本地分支](#delete_local_branch) 的命令在前面已经介绍过了，现在我们来删除刚才用于演示的 <code>local-dev</code> 和 <code>local-cache</code> 本地分支，以及它们对应的远程仓库中的分支：
```
$ git branch -d local-dev
$ git push origin --delete local-dev
$ git branch -d local-cache
$ git push origin --delete remote-cach
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/team
  remotes/origin/test
  remotes/origin/tmp
```
可以看到远程库中的 <code>local-dev</code> 和 <code>remote-cache</code> 分支都被删除掉了。

#### 跟踪分支
从一个远程跟踪分支检出一个本地分支会自动创建一个叫做 “跟踪分支”（有时候也叫做 “上游分支”）。跟踪分支是与远程分支有直接关系的本地分支。如果在一个跟踪分支上输入 <code>git pull</code>，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

前面我们已经知道，当克隆一个仓库时，它会自动地创建一个跟踪 <code>origin/master</code> 的 <code>master</code> 分支。然而，如果你愿意的话可以设置其他的跟踪分支 - 跟踪其他远程仓库上的分支或者不跟踪 <code>master</code> 分支。

假设有个小组的管理人员在本地创建了 <code>team</code> 分支并将其推送了远程仓库，现在指定小组内的所有人都要在这个 <code>team</code> 分支上进行工作。当管理人员将 <code>team</code> 分支推送到远程仓库后，其他人通过 <code>git fetch</code> 从服务器上抓取数据时，他们会在本地生成一个 <code>origin/team</code> 远程分支，指向服务器中 <code>team</code> 分支的引用。

需要特别注意是当抓取到新的远程跟踪分支时，本地并不会自动生成一份可编辑的副本（拷贝）。换句话说，系统不会在本地创建一个 <code>team</code> 分支，只有一个不可修改的 <code>origin/team</code> 指针。那么我们如何创建一个跟踪 <code>team</code> 分支的本地分支呢？有三种实现方式：

1.第一种
```
$ git checkout --track origin/team
Branch team set up to track remote branch team from origin.
Switched to a new branch 'team'
```

2.第二种
```
$ git checkout -b team origin/team
Branch team set up to track remote branch team from origin.
Switched to a new branch 'team'
```
当然了，我们可以将本地分支与远程分支设置为不同的名字，只需要将 <code>team</code> 换成我们需要的名字即可。比如运行 <code>git checkout -b my-team origin/team</code>，现在 <code>my-team</code> 被设置为用来跟踪远程仓库中的 <code>team</code>，本地分支 <code>my-team</code> 会自动从 <code>origin/team</code> 拉取数据。

3.第三种
```
$ git checkout -b team
Switched to a new branch 'team'
$ git branch -u origin/team
Branch team set up to track remote branch team from origin.
```
我们可以先创建一个本地分支，然后切换到该分支后执行 <code>git branch -u remote-name/branch-name</code> 或者 <code>git branch -\-set-upstream-to=remote-name/branch-name</code> 命令来显示地设置让当前本地分支追踪某个上游分支。通过这个命令我们还可以修改正在跟踪的上游分支。比如我们刚才让本地的 <code>team</code> 追踪上游的 <code>team</code>，现在我们在 <code>team</code> 分支上的所有工作都完成了并且合并到了主分支 <code>master</code> 上面，后续的工作基于 <code>master</code> 分支继续进行就可以了，那么我们现在就可以使用 <code>git branch -u origin/master</code> 让 <code>team</code> 追踪主分支 <code>master</code>，这样我们就可以在 <code>team</code> 分支上继续工作了。

想要查看设置的所有跟踪分支，可以使用<code>git branch -vv</code> 命令。这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支，本地分支是否是领先、落后或是都有。

示例：
```
$ git branch -vv
  dev    2617b13 [origin/dev] Merge branch 'test'
  master 2617b13 [origin/master] Merge branch 'test'
* team   2617b13 [origin/team: behind 1] Merge branch 'test'
```
可以看到 <code>dev</code> 分支正在跟踪 <code>origin/dev</code> 分支并且是最新的。也能看到 <code>master</code> 分支正在跟踪 <code>origin/master</code> 分支并且也是最新的。最后可以看到 <code>team</code> 分支正在跟踪 <code>origin/team</code> 分支并且落后1，意味着服务器上还有一次提交没有合并到本地。如果我们看到 <code>ahead x</code> 之类的提交信息，意味着本地有x次提交还没有推送到服务器上。

需要注意的是这些数字的值只是来自于你从每个服务器上最后一次抓取的数据。这个命令并没有连接服务器，它只会告诉你关于本地缓存的服务器数据。如果想要统计最新的领先与落后数字，需要先运行 <code>git fetch -\-all</code> 命令后再运行查看跟踪分支的命令。

#### 拉取数据
使用 <code>git fetch</code> 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容，它只会获取数据然后让你自己合并。然而，有一个命令叫作 <code>git pull</code>， 在大多数情况下它的含义是一个 <code>git fetch</code> 紧接着一个 <code>git merge</code> 命令。如果你像之前章节中演示的那样设置好了一个跟踪分支，不管它是显式地设置还是通过 <code>clone</code> 或 <code>checkout</code> 命令创建的，<code>git pull</code> 都会查找当前分支所跟踪的服务器与分支，从服务器上抓取数据然后尝试合并入那个远程分支。

由于 <code>git pull</code> 的魔法经常令人困惑，所以通常单独显式地使用 <code>fetch</code> 与 <code>merge</code> 命令会更好一些。

### 变基
在 Git 中整合来自不同分支的修改主要有两种方法：<code>merge</code> 以及 <code>rebase</code>。关于 <code>merge</code> 以及 <code>rebase</code> 命令的原理请参考文档 [变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)。我们已经在前面 [合并分支](#merge_branches) 章节介绍过了如何使用 <code>merge</code> 命令，现在开始介绍如何使用 <code>rebase</code> 命令。还是以实例操作来演示，这样方便直观，便于理解和学习。

1.首先切换到 <code>master</code> 分支，在 <code>log.md</code> 文件上添加一行说明性文字，然后将修改提交到仓库中：
```
$ git checkout master
$ git add log.md
$ git commit -m "Generate a commit log in repository for 'git rebase' test"
$ git push origin master
$ git log --oneline -2
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
2617b13 Merge branch 'test'
```
稍后我们将以 <code>13f83a3</code> 为节点来演示 <code>rebase</code> 操作。

2.新建一个 <code>rebase_demo</code> 分支，切换到该分支后按照如下步骤操作：
```
$ git checkout -b rebase_demo
$ touch rebase_on_rebase_demo.md
$ git add rebase_on_rebase_demo.md
$ git commit -m "Add rebase_on_rebase_demo.md file on rebase_demo branch"
$ git log --oneline -2
176ec79 Add rebase_on_rebase_demo.md file on rebase_demo branch
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
```
现在我们在 <code>rebase_demo</code> 分支上有了一个 <code>176ec79</code> 提交。

3.切回 <code>master</code> 分支，按如下步骤操作：
```
$ git checkout master
$ touch rebase_on_master.md
$ git add rebase_on_master.md
$ git commit -m "Add rebase_on_master.md file in master branch"
$ git log --oneline -2
6340e0d Add rebase_on_master.md file in master branch
13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
```
现在我们在 <code>master</code> 分支上有了一个 <code>6340e0d</code> 提交。

我们知道 <code>176ec79</code> 和 <code>6340e0d</code> 的共同祖先是 <code>13f83a3</code>。

使用 <code>merge</code> 命令将 <code>rebase_demo</code> 分支上的修改合并到 <code>master</code> 分支时，Git 会对 <code>176ec79</code> 、<code>6340e0d</code> 以及  <code>13f83a3</code> 进行三方合并，生成一个新的快照（并提交）。除了通过合并整合分叉的历史，还有一种方法就是使用 <code>rebase</code> 命令。方法如下：首先提取在 <code>176ec79</code> 中引入的补丁和修改，然后在 <code>6340e0d</code> 的基础上应用一次。 在 Git 中，这种操作就叫做 **变基**。你可以使用 <code>rebase</code> 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。命令操作如下：
```
$ git checkout rebase_demo
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Add rebase_on_rebase_demo.md file on rebase_demo branch
```

它的原理是首先找到这两个分支（即当前分支 <code>rebase_demo</code>、变基操作的目标基底分支 <code>master</code>）的最近共同祖先 <code>13f83a3</code>，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底 <code>6340e0d</code>，最后以此将之前另存为临时文件的修改依序应用。

现在回到 <code>master</code> 分支进行一次快进合并：
```
$ git checkout master
$ git merge rebase_demo
$ git log --oneline --graph -3
* 827ff4b Add rebase_on_rebase_demo.md file on rebase_demo branch
* 6340e0d Add rebase_on_master.md file in master branch
* 13f83a3 Summary: Generate a commit log in repository for 'git rebase' test
```

此时，<code>827ff4b</code> 指向的快照就和使用 <code>merge</code> 命令最终生成的快照一模一样了。***这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。***上面我们用命令查看经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。

一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个其他人维护的项目贡献代码时。在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到 <code>origin/master</code> 上，然后再向主项目提交修改。这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。

请注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

关于变基的一些更高级的操作请阅读 [变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA) 中更有趣的例子。必须记住，使用变基必须要准守一条准则：**不要对在你的仓库外有副本的分支执行变基**。如果你遵循这条金科玉律，就不会出差错。否则，人民群众会仇恨你，你的朋友和家人也会嘲笑你，唾弃你。


## 参考文章
[Git 官方中文手册](https://git-scm.com/book/zh/v2)
