---
layout: post
category : 转载
tagline: "Supporting tagline"
tags : [Git, 代码管理, 协作]
---
Git Cheat Sheet Chinese
========================
> <em><sub>----------------戴维营教育整理--------------------</sub>

###索引
* [创建](#创建)
* [本地修改](#本地修改)
* [搜索](#搜索)
* [提交历史](#提交历史)
* [分支与标签](#分支与标签)
* [更新与发布](#更新与发布)
* [合并与重置](#合并与重置)
* [撤销](#撤销)

---

###创建

复制一个已创建的仓库:
```bash
$ git clone ssh://user@domain.com/repo.git
```

创建一个新的本地仓库:
```bash
$ git init
```

---
###本地修改

显示工作路径下已修改的文件：
```bash
$ git status
```

显示与上次提交版本文件的不同：
```bash
$ git diff
```

把当前所有修改添加到下次提交中：
```bash
$ git add .
```

把对某个文件的修改添加到下次提交中：
```bash
$ git add -p <file>
```

提交本地的所有修改：
```bash
$ git commit -a
```

提交之前已标记的变化：
```bash
$ git commit
```

附加消息提交：
```bash
$ git commit -m 'message here'
```

Commit to some previous date:
```bash
git commit --date="`date --date='n day ago'`" -am "Commit Message"
```

修改上次提交<br/>
<em><sub>Don't amend published commits!</sub></em>
```bash
$ git commit --amend
```
把当前分支中未提交的修改移动到其他分支
```bash
git stash
git checkout branch2
git stash pop
```

###搜索

从当前目录的所有文件中查找文本内容：
```bash
$ git grep "Hello"
```

在某一版本中搜索文本：
```bash
$ git grep "Hello" v2.5
```

---
###提交历史

从最新提交开始，显示所有的提交记录（显示hash， 作者信息，提交的标题和时间）：
```bash
$ git log
```

显示所有提交（仅显示提交的hash和message）：
```bash
$ git log --oneline
```

显示某个用户的所有提交：
```bash
$ git log --author="username"
```

显示某个文件的所有修改：
```bash
$ git log -p <file>
```

谁，在什么时间，修改了文件的什么内容：
```bash
$ git blame <file>
```

---
###分支与标签

列出所有的分支：
```bash
$ git branch
```

切换分支：
```bash
$ git checkout <branch>
```

基于当前分支创建新分支：
```bash
$ git branch <new-branch>
```

基于远程分支创建新的可追溯的分支：
```bash
$ git branch --track <new-branch> <remote-branch>
```

删除本地分支:
```bash
$ git branch -d <branch>
```

给当前版本打标签：
```bash
$ git tag <tag-name>
```

---
###更新与发布

列出对当前远程端的操作：
```bash
$ git remote -v
```

显示远程端的信息：
```bash
$ git remote show <remote>
```

添加新的远程端：
```bash
$ git remote add <remote> <url>
```

下载远程端版本，但不合并到HEAD中：
```bash
$ git fetch <remote>
```

下载远程端版本，并自动与HEAD版本合并：
```bash
$ git remote pull <remote> <url>
```

将远程端版本合并到本地版本中：
```bash
$ git pull origin master
```

将本地版本发布到远程端：
```bash
$ git push remote <remote> <branch>
```

删除远程端分支：
```bash
$ git push <remote> :<branch> (since Git v1.5.0)
or
git push <remote> --delete <branch> (since Git v1.7.0)
```

发布标签:
```bash
$ git push --tags
```

---
###合并与重置

将分支合并到当前HEAD中：
```bash
$ git merge <branch>
```

将当前HEAD版本重置到分支中:<br>
<em><sub>Don't rebase published commit!</sub></em>
```bash
$ git rebase <branch>
```

退出重置:
```bash
$ git rebase --abort
```

解决冲突后继续重置：
```bash
$ git rebase --continue
```

使用配置好的merge tool 解决冲突：
```bash
$ git mergetool
```

在编辑器中手动解决冲突后，标记文件为`已解决冲突`
```bash
$ git add <resolved-file>
```
```bash
$ git rm <resolved-file>
```

---
###撤销

放弃工作目录下的所有修改：
```bash
$ git reset --hard HEAD
```

移除缓存区的所有文件（i.e. 撤销上次`git add`）:
```bash
$ git reset HEAD
```

放弃某个文件的所有本地修改：
```bash
$ git checkout HEAD <file>
```

重置一个提交（通过创建一个截然不同的新提交）
```bash
$ git revert <commit>
```

将HEAD重置到上一次提交的版本，并放弃之后的所有修改：
```bash
$ git reset --hard <commit>
```

将HEAD重置到上一次提交的版本，并将之后的修改标记为未添加到缓存区的修改：
```bash
$ git reset <commit>
```

将HEAD重置到上一次提交的版本，并保留未提交的本地修改：
```bash
$ git reset --keep <commit>
```

---
转载自[https://github.com/ArslanBilal/Git-Cheat-Sheet](https://github.com/ArslanBilal/Git-Cheat-Sheet)
