---
title: Git常用命令总结
date: 2018-03-19 14:56:37
tags:
    - Git
categories:
---
添加文章描述内容

<!-- more -->

### Git 分支
1.显示所有本地分支：当前分支以<code>*</code>高亮
```
git branch
```

2.显示所有远程分支
```
git branch -r
```

3.显示所有本地和远程分支
```
git branch -a
```

4.删除本地分支
```
git branch -d branch_name
```

5.删除远程分支
```
git push origin --delete branch_name
git push origin :branch_name
```
第一种方式是 Git 官方手册介绍的删除远程分支的方法，官方手册对此命令有一段描述如下说明：“基本上这个命令做的只是从服务器上移除这个指针。 Git 服务器通常会保留数据一段时间直到垃圾回收运行，所以如果不小心删除掉了，通常是很容易恢复的。”

第二种方式是网上介绍的另一种删除分支的方法，也可以删除远程分支，但不了解该命令是否有暂存数据的功能，所以最好使用官方介绍的删除分支方法。

6.创建一个新的分支
```
git branch branch_name
```

7.切换到指定分支
```
git checkout branch_name
```

8.新建一个分支并切换到新建的分支
```
git checkout -b branch_name
```


参数标注：
- branch_name : 操作的分支名称


### 参考文章
[Git 官方中文手册](https://git-scm.com/book/zh/v2)
