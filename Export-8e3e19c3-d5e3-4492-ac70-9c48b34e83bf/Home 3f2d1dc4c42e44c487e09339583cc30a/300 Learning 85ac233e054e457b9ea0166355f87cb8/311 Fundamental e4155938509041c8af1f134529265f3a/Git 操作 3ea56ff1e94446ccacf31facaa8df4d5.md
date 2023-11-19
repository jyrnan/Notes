# Git 操作

[](http://git.konka.com/zhangxin/study/blob/master/Gerrit%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/gerrit%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.md)

Gerrit操作，需要切换内网查看

[Gerrit提交时候报错“missing Change-Id in message footer”](Git%20%E6%93%8D%E4%BD%9C%203ea56ff1e94446ccacf31facaa8df4d5/Gerrit%E6%8F%90%E4%BA%A4%E6%97%B6%E5%80%99%E6%8A%A5%E9%94%99%E2%80%9Cmissing%20Change-Id%20in%20message%20footer%E2%80%9D%201413c1afb09743d3850d135274983457.md)

[Git工作流中常见的三种分支策略：GitFlow、GitHubFlow以及GitLabFlow](Git%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E4%B8%89%E7%A7%8D%E5%88%86%E6%94%AF%E7%AD%96%E7%95%A5%EF%BC%9AGitFlow%E3%80%81GitHubFlow%E4%BB%A5%E5%8F%8AGitLabFlow%20e2db9453df9e4fa5a9a276157b4b0cd6.md)

[git-flow 的工作流程 | Learn Version Control with Git](git-flow%20%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%20Learn%20Version%20Control%20with%20Git%2031f4551556f846b6ab451bd04b6de7a6.md)

### 一个看起来很不错的Git教学网站，还没完全看完

[什么是版本控制？](https://www.git-tower.com/learn/git/ebook/cn/command-line/basics/what-is-version-control#start)

# 基本操作

```bash
$ git init
$ git add .
$ git remote add origin [github-URL]
$ git commit -m "Initial Commit"
$ git tag 1.0.0
$ git push origin master --tags
```

- 可以用 `Git remote -v` 来查看远端节点的信息
- `git remote rename`  来修改远端节点的名字。

### **补充：**

<aside>
💡 **git remote add orgin 这里的origin代表是一个远端位置名称name，也就是代表后面的url地址。不要和远端上的分支搞混了**

</aside>

# **Git代理设置方式**

只对github.com

```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
```

取消代理

```bash
git config --global --unset http.https://github.com.proxy
```

# 关于Reset

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

# 关于Git Clone地址

你也许还注意到，GitHub给出的地址不止一个，还可以用`https://github.com/michaelliao/gitskills.git`这样的地址。实际上，Git支持多种协议，默认的`git://`使用ssh，但也可以使用`https`等其他协议。

使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用`ssh`协议而只能用`https`。

# 分支操作

## 使用分支基本操作：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

## 分支策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

所以，团队合作的分支看起来就像这样：

[https://www.liaoxuefeng.com/files/attachments/919023260793600/0](https://www.liaoxuefeng.com/files/attachments/919023260793600/0)

## 合并分支

### 基本方法:

把hotfix分支合并到master分支

- 先切换到目标分支，例如main。 `git checkout m`aster
- 再把要合并到分支合并到当前分支 `git merge` hotfix

### 三方合并：

最关键和不太好理解的是三方合并的merge！

分支历史从一个更早的地方开始分叉开来（diverged）。`master`分支所在提交并不是`iss53`分支所在提交的直接祖先，Git需要做一些额外的工作。遇到这种情况Git会使用分支末端所指的快照（`C4`
和`C5`）以及这两个分支的祖先（`C2`），做一个三方合并(3-way merge），三方合并的结果是产生一个新的快照并且自动创建一个新的提交指向它，被称为一次合并提交，其特别之处在于它有不止一个父提交。

![Untitled](Git%20%E6%93%8D%E4%BD%9C%203ea56ff1e94446ccacf31facaa8df4d5/Untitled.png)

**需要理解注意的几点**

- 是从iss53合并到master，所以新产生的提交C6是属于master上的。可以理解成在C2的基础上，同时应用了master上的C4和iss53上C3和C5三个提交修改形成的C6。
- 因为两条分支都对C2产生了的的提交，所以可能会冲突，参考[下面解决方式](Git%20%E6%93%8D%E4%BD%9C%203ea56ff1e94446ccacf31facaa8df4d5.md)
- iss53的头依然是C5，如果切换过来iss53，Head将指向C5，如果此时继续修改提交，产生C7，可以直接再次将iss53合并到master中，会再次进行一次合并提交，产生C8。因为这个过程中master的C6后并没有新的提交，所以C8必定不会和C7冲突。出现的图就是如下情况。

[https://www.liaoxuefeng.com/files/attachments/919023260793600/0](https://www.liaoxuefeng.com/files/attachments/919023260793600/0)

## 解决分支合并冲突处理

### 出现冲突

如果你在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改(文本文件是用行号标识，例如同一行），Git在合并时将产生冲突（conflict），不能自动合并。

- 如果是同一个文本文件，Git会自动把冲突内容进行合并到文件中，并会用符号标示出来。此时，Git做了合并，但是没有自动创建一个新的合并提交。
    
    ```bash
    $ git merge iss53
    Auto-merging README.md
    CONFLICT (content): Merge conflict in README.md // Git把冲突内容合并到这个文件
    Automatic merge failed; fix conflicts and then commit the result.
    ```
    
    - 使用`git status`命令查看因包含冲突而处于未合并（unmerged）状态的文件。
        
        ```bash
        // 1.查看哪些文件产生了合并冲突。
        $ git status
        On branch master
        You have unmerged paths.
          (fix conflicts and run "git commit")
          (use "git merge --abort" to abort the merge)
        
        Unmerged paths:
          (use "git add <file>..." to mark resolution)
        
        	both modified:   README.md
        
        no changes added to commit (use "git add" and/or "git commit -a")
        ```
        
    - 使用`git diff`查看具体冲突部分。
        
        ```bash
        // 2.查看冲突明细。
        $ git diff
        diff --cc README.md
        index 582c321,6630dce..0000000
        --- a/README.md
        +++ b/README.md
        @@@ -1,2 -1,4 +1,8 @@@
          Git is a distributed version control system.
        ++<<<<<<< HEAD
         +Git is a free software distributed under the GPL.
        ++=======
        + git is a free software distributed under the GPL.
        + Creating a new branch is quick.
        + Git has a mutable index called stage.
        ++>>>>>>> iss53
        ```
        

### 解决冲突

Git会暂停下来，等待你去解决合并过程中产生的冲突。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容。可以直接在git自动合并内容的冲突文件中编辑成最终的内容。

- 解决冲突后，使用`git add`添加刚才修改的冲突文件、
- 使用`git commit`提交更改。（这里不是提交新的commit，而是把刚才未完成的合并继续完成。随后会提示给这次合并创建备注文字；但严格来说也可以是是一次提交，因为merge其实就是创建一个新的commit）

用`git log --graph`命令可以看到分支合并图。

## Bug分支的处理方法： stash和cherry-pick

### 这是一种常见状况

main分支发布，dev分支进行开发，目前已经进行了一段工作，但是急需修复`main`分支一个bug。

- 用`git stash`保留好dev上的工作
- 从`main`分支新开分支上修复，就从`main`创建临时分支，在临时分支修复bug并提交，然后删除临时分支 。此时main分支上有了新的commit，命名`commit_a`。
- 回到dev分支继续，但此时dve是从main早期分出来的，也就是说没有包含main上修复bug的commit_a，此时可以用`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：Git自动给dev分支做了一次提交，注意这次提交的commit是`1d4b803`，它并不同于`main`的`commit_a`，因为这两个commit只是改动相同，但确实是两个不同的commit。用`git cherry-pick`，我们就不需要在dev分支上手动再把修bug的过程重复一遍。

```swift
$ git branch
* dev
  main
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

- 然后再`git stash pop` 来恢复原来再dev分支上进行的工作

> 先`git stash pop` 还是应该先进行`cherry-pick`从main分支上获取修复bug提交的内容实验过应该都可以
> 

### 小结

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场；

在master分支上修复的bug，想要合并到当前dev分支，可以用`git cherry-pick <commit>`命令，把bug提交的修改“复制”到当前分支，避免重复劳动。

# 多人协作

## 多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

## 小结

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

```
// 1.查看哪些文件产生了合并冲突。
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

---

[Reference](../../UniLabel%2030f43971c25f4cbea695984a0b8c1b6d/Reference%20dc0cb9e2b65640b7b4f01bf31bc8c233.md)