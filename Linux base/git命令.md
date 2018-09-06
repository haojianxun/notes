# git命令

`git ls-files` :列出文件  -s表示列出stage area区域的内容

`git cat-file`  查看文件    可以带-p选项  表示美观排版显示

`git hash-object` :计算文件的hash码

`git write-tree` :根据当前索引的内容创建树对象



## add/rm/mv命令

`git add` 暂存文件

​	git ls-files:

​		默认显示索引中的文件列表的原始文件名

​		-s : 显示暂存的文件信息:权限,对象名,暂存号和原始文件名

​		-o : 显示为被追踪的文件

`git rm` 

​	git rm:删除工作目录的文件,及索引的映射

​	git rm --cached :只删除索引中的映射

`git mv` 

​	git mv ;改变工作目录的文件名 和索引中的映射





`git diff` 比较提交 索引及工作目录

`git reset` 撤销此前的操作

​	--soft :将HEAD引用指向给定的提交,但是不影响索引和工作目录

​	--mixed:将HEAD引用指向给定的提交并将索引内容改变为指定提交的快照,但是不改变工作目录

​	--hard:将HEAD引用指向给定的提交,将索引内容改变为指定提交的快照,并改变工作目录中的内容反映提交的内容



`git branch` 列出,创建以删除分支

​	git branch BRANCH_NAME[START_COMMIT] 

​	git branch -d BRANCH_NAME  -d代表删除

​	--list 列出分支,  *代表当前分支

` git show-branch` 查看分支及其相关的提交



`git checkout` 

​	git checkout \<branch\> :检出分支

​	

git remote命令 管理远程仓库

git pull 取回远程服务器更新 而后与本地的指定分支合并

​	git pull 远程主机名 远程分支名 : 本地分支名

gut push 将本地的更新推送到远程主机

​	git push 远程主机名 本地分支名:远程分支名



#### 操作标签

阅读: 151837

------

如果标签打错了，也可以删除：

```
$ git tag -d v0.1
Deleted tag 'v0.1' (was e078af9)

```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

如果要推送某个标签到远程，使用命令`git push origin <tagname>`：

```
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
 * [new tag]         v1.0 -> v1.0

```

或者，一次性推送全部尚未推送到远程的本地标签：

```
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 554 bytes, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:michaelliao/learngit.git
 * [new tag]         v0.2 -> v0.2
 * [new tag]         v0.9 -> v0.9

```

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```
$ git tag -d v0.9
Deleted tag 'v0.9' (was 6224937)

```

然后，从远程删除。删除命令也是push，但是格式如下：

```
$ git push origin :refs/tags/v0.9
To git@github.com:michaelliao/learngit.git
 - [deleted]         v0.9

```

要看看是否真的从远程库删除了标签，可以登陆GitHub查看。





- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。