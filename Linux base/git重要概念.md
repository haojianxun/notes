# git重要概念

git

- 工作区:working directory

  就是你所在的目录

- 暂存区:staging area

  Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

- 版本区:repository

  工作区有一个隐藏目录`.git`，是Git的版本库。



![](http://www.liaoxuefeng.com/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)

git配置文件:git config

​	仓库特有:REPO/.git/config

​	全局:~/.gitconfig, --global

​	系统:/etc/git/gitconfig  --system

​	user.name user.email

git仓库

​	索引:暂存

​	对象库:版本库

git对象类型:

​	块(blob)对象:文件的每个版本表现为一个块

​	树(tree)对象:一个目录代表一层目录信息

​	提交对象:用于保存版本库一次变化的元数据,包括作者,邮箱,提交日期.日志,每个提交对象都有一个目录树对象

​	标签对象:用于给一个特定对象一个易读的名称



git中文件分类

- 已追踪的(tracked):已经在版本库中,或者已经使用`git add` 命令添加到索引的文件
- 被忽略的(igored):在版本库中通过"忽略文件列表"明确声明为被忽略的文件
- 未追踪的(untracked):上述两类之外的其他文件

## git提交

`git commit` 

`git log` :查看提交文件



提交的标识:

​	引用:ID reference, SHA1 ,绝对提交名

​	符号引用:symbolic reference,symref

​	

refs/heads/REF:本地特性分支名称

refs/remotes/REF 远程跟踪分支名称

refs/tags/REF 标签名



git会自动维护几个特定目的的特殊符号引用

​	HEAD:始终指向当前分支的最近提交,或检测出其他分支时,目标分支的最近提交

​	ORIG_HEAD:合并操作时,新生成的提交之前的提交保存在此引用中

​	FETCHED_HEAD:

​	MERGE_HEAD:合并操作时,其他分支的上一次提交



## git分支

分支命令法则

- 可以使用/,但是不能以/结尾
- 不能以-开头
- 以位于/后面的组件,不能以.开头
- 不能使用连续的....
- 不能使用空白字符
- 不能使用^ , ~ , ? , * , [等

必须唯一,分支名字的名字始终指向目标分支的最近一次提交



分支合并

​	合并基础:要合并的分支的最近一次的共同提交

​	我们的版本:当前分支的最近一次提交

​	他们的版本:要合并进来的分支的最近一次提交

无冲突合并

```
git checkout master
git status
git merge BRANCH_NAME
git log --graph --pretty=oneline --abbreb-commit
```

有冲突合并

```

```



​	