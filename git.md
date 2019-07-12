## Git中的`^`和`~` ##

把Git中的提交作为节点来看待，那么一个节点可以有0个（初始提交），一个或两个父节点（两个父节点合并生成该节点）。`^`和`~`都可以用来指代某个节点的父节点，但他们有所区别（后面跟数字时的含义不同）：

 - `^`：可用于表达合并生成节点的父节点，`^1`代表当前节点的第一父节点，即当前节点所在分支上的父节点，也是合并操作中合并到的节点；`^2`代表当前节点的第二父节点，即合并操作中被合并的节点；`^3`和以后的带数字的`^`没有意义，因为不可能再有其他父节点；没有后跟数字的`^`等同于`^1`；
 - `~`：只能用于表达当前节点所在分支上的父节点，数字代表的是向前追溯的节点数，`~1`代表向前追溯一个节点，即直接父节点，等同于`^`；`~2`代表前溯2个节点，即祖父节点，等同于`^^`，依次类推；未后跟数字的`~`等同于`~1`。

## 本地仓库的4种状态 ##

- clean：本地仓库与远程仓库同步，工作目录中没有任何改动；
- unstaged：紧接clean状态，但更改了工作目录中的内容；
- staged：把unstaged状态中更改的内容通过`git add`或`git stage`命令记录到了本地仓库的待提交区；
- committed：把待提交区的所有内容通过`git commit`命令提交到了本地仓库。如果之后执行了push操作，则本地仓库又进入了新的clean状态。

## 分支 ##

分支有三种：

- 本地分支：比如`master`
- 本地的远程分支映射：比如`origin/master`，相当于指向远程分支的本地快捷方式
- 远程分支：比如`remote/origin/master`

## Git中的` `，`..`和`...` ##

- 在`git rev-list`中代表范围：
	- `git rev-list branch1 branch2`：branch1和branch2中所有的节点；
	- `git rev-list branch1..branch2`：branch1和branch2共同的父节点（排除）和branch2之间的所有的节点；
	- `git rev-list branch1...branch2`：branch1和branch2共同的父节点（排除）和branch1之间的所有节点+和branch2之间的所有的节点；
- 在`git diff`中代表比较对象：
	- `git diff branch1 branch2`：branch1最新节点和branch2最新节点的对比；
	- `git diff branch1..branch2`：同上，branch1最新节点和branch2最新节点的对比；
	- `git diff branch1...branch2`：branch1和branch2共同的父节点和branch2最新节点的对比；

## `fetch`和`pull`的区别 ##

首先得了解远程仓库，本地仓库和工作目录是不同的概念，尤其是本地仓库和工作目录的区别，本地仓库指的是`.git`目录中保存的仓库信息，而工作目录是指项目目录中除`.git`目录外的的其他目录和文件。远程仓库则包含了远程仓库信息和最新的项目文件，因此并没有远程工作目录这一说法。

- `fetch`命令根据远程仓库的最新信息更新了本地仓库，但工作目录并不受影响，也就是说此时的工作目录中并没有包含远程仓库的任何修改，但本地仓库中却包含了这些信息，因此此时，本地仓库和远程仓库是同步的，工作目录则与前两个仓库不同步（除非最新的更新都是针对git元数据的修改，而文件结构和内容没有任何改变）
- `pull`命令则是同时更新本地仓库和工作目录，此时远程仓库，本地仓库和工作目录都是同步的。

![Git 常用命令示意](https://i0.wp.com/eissanematollahi.com/wp-content/uploads/2018/08/git-workflow.png?w=640&ssl=1)

## 工作日常Git命令 ##

### 在本地创建新的仓库 ###

		git init
		#在此目录下进行操作
		git add -A
		git commit -m "first commit"
		git remote add origin https://github.com/youraccount/repo.git
		git push -u origin master

### 从远程克隆仓库 ###

		git clone https://github.com/youraccount/repo.git

### 添加远程仓库 ###

		git remote add local-reference-name https://github.com/account/repo.git

### 查看当前所有的远程仓库 ###

		git remote -v

### 更改远程仓库 ###

		#origin是远程仓库的默认引用名称
		git remote set-url origin https://github.com/account/reps.git

### 查看远程仓库的信息 ###

		git remote show origin

### 查看仓库日志 ###

		#可以在.gitconfig文件中为以下命令建立别名
		git log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --branches 

### 查看所有本地和远程分支 ###

		git branch -a
		#下面的命令显示所有本地分支与远程分支的关系
		git branch -vv

### 创建本地分支 ###

		git checkout -b new-branch-name origin/branch

### 重命名本地分支 ###

		git checkout -m old-branch-name new-branch-name

### 切换分支 ###

		git checkout branch-name

### 更新当前分支的内容 ###

		#更新本地仓库
		git fetch
		#使用前面建立的log别名查看更新日志
		git lg
		#确定对当前工作产生不良影响后更新工作目录
		git pull

### pull失败 ###

很可能是本地分支没有和远程分支关联，使用如下命令创建关联：

		git branch set-upstream-to=origin/remote-branche-name local-branch-name

### 提交本地分支 ###

		#如果远程仓库中没有dev分支，会自动创建该分支remote/origin/dev
		git push origin dev
		#上面的命令不会把远程分支与当前分支关联
		#可以在第一次提交时使用如下命令在创建远程分支的同时创建其在本地的映射并与当前分支关联
		git push -u origin dev

### 删除本地分支 ###

		git branch -d local-branch
		#如果该分支还没有被合并，则会因为丢失分支中所有提交而报错。可以使用-D参数强制删除
		git branch -D local-branch
		#删除本地分支不会影响与之关联的远程分支的本地映射
		#删除远程分支的本地映射（不会影响远程分支）
		git branch -d -r origin/branch

### 把本地分支合并到远程分支 ###

		#推荐使用rebase方式，可以获得更清爽的历史记录
		git pull --rebase origin/branch-name

### 在当前分支工作可以提交前需要在其他分支上工作 ###

这种情况是指在当前分支的工作还没有完成，还无法提交（每一次提交都应该有意义，而不是为了临时保存工作），此时却需要转到其他分支工作。为了不放弃已有的修改（正常途径下，不提交就不会保存修改），又不希望修改出现在其他分支中，需要使用stash命令把当前的工作暂存起来。完成暂存后，工作目录会恢复到修改之前的clean状态，此时就可以在新的分支上工作。

		#保存当前分支中的工作
		git stash
		#默认只有已被跟踪的文件（不管是否目前已被放入待提交区）才会被暂时保存
		#如需保存未跟踪的文件
		git stash -u
		#如需保存包括被忽略文件的所有文件
		git stash --all
		#查看当前暂存的工作列表
		git stash list
		#查看某个暂存的工作的内容，n代表该工作在列表中的位置
		git stash show stash@{n}
		#恢复指定的暂存工作
		git stash apply stash@{n}
		#不指定暂存工作，则默认恢复第一个
		#下面命令相当于git stash apply stash@{0}
		git stash apply
		#恢复暂存工作后，暂存的工作不会被删除，可以使用如下命令手工删除
		git stash drop stash@{n}
		#下面命令可以恢复并删除暂存的工作
		git stash pop stash@{n}

### 需要暂存部分文件 ###

		#注意add的文件都是不想被暂存的文件
		git add filename1, filename2
		#暂存剩下的文件
		git stash -keep-index
		#将add的文件恢复到unstaged状态
		git reset HEAD

### 删除远程分支 ###

		git push origin --delete branch-name

### 查看工作目录的状态 ###

		#-s 表示简单描述
		git status -s	

### 删除所有未追踪的文件（untracked） ###

		#只影响工作目录，对本地仓库没有影响
		#-d：包含目录
		#-f：强制删除所有内容，包括.git目录
		git clean -df
		#如果希望在删除前进行确认，可以添加-n参数，只显示准备删除的内容而不会真的执行操作
		git clean -dfn

### 在保留其他修改的同时丢弃对特定文件的修改 ###

		git checkout filename
		#从其他分支或之前的提交中签出文件的不同版本
		git checkout other-branch filename

###  回溯到某个提交历史版本，该版本之后所有的修改都处于unstaged状态 ###
		
		#只影响本地仓库，对工作目录没有影响
		git reset HEAD

### 回溯到某个提交历史版本，该版本之后所有的修改都处于staged状态（待提交区） ###

		#只影响本地仓库，对工作目录没有影响
		git reset --soft HEAD
		git reset --soft {some-start-point-hash}

### 撤销之前的提交 ###

		#自动将回退的文件提交到仓库，需要写一个新的提交信息
		git revert
		#如果只是想预览一下结果，可以使用-n参数先不提交
		git revert -n

### 在推送到远程仓库前，为上一次本地提交补充内容 ###

		#只影响本地仓库，对工作目录没有影响
		git commit --amend 
		#如果不需要重新编辑提交信息，可以添加--no-edit参数
		git commit --amend --no-edit
		#如果已经推送到远程仓库，并且是多个人合作开发，最好还是在修正错误后创建新的提交

### 发现一团糟，决定放弃自上次pull以来的一切修改 ###

		git reset --hard HEAD
		#上面的操作会把对本地目录内容的修改全部变为unstaged，因此还需要执行下面的命令来删除这些修改
		git clean -df

### 查看工作目录的具体更改内容 ###

		#如果文件还没有放入待提交区
		git diff 
		#如果文件已经放入待提交区
		git diff --cached

### 查看某个具体文件的更改及其作者 ###

		git blame filename
		#添加-w参数忽略对空白的修改
		git blame -w 
		#其他命令如diff也可以使用-w参数

### 查看两个分支之间的差别 ###

		#查看branch1和branch2之间的差别
		git diff branch1 branch2 
		#等同于
		git diff branch1..branch2
		#查看branch1和branch2共同的父节点和branch1之间的差别
		git diff branch2...branch1
		#查看某个文件在当前分支和另一分支之间的差别
		git diff other-branch filename

### 查找并恢复丢失的提交 ###

		#显示HEAD移动的历史记录
		git reflog --decorate
		#在显示的列表中查看提交丢失前HEAD的位置，比如HEAD@{n}
		#签出该位置的仓库
		git checkout -b branch-name HEAD@{n}

### 处理冲突 ###

当合并发生冲突需要人工处理时，发生冲突的文件内容格式如下：

		<<<<<<< HEAD (HEAD -> master)
		本地内容
		=======
		远程内容
		>>>>>>> 远程更改的提交对应的SHA-1码 (origin/master)

		#使用本地内容
		git checkout --ours filename
		#使用远程内容
		git checkout --theirs filename
		#或者手工编辑内容，解决冲突

		#中止此次尝试，重新来过
		git merge --abort

		#重新将修改后的文件加入待提交区并提交
		git add filename
		git commit -m "proper commit message"
		git push

注意，如果是rebase，则两部分内容的位置是相反的。

		git pull --rebase

### 让本地分支和远程主分支同步时，发现主分支已经有新的内容，需要更新 ###

推荐使用rebase，因为merge会把主分支的提交和本地提交历史混在一起再拷贝一遍。

		git rebase master
		#如果之前曾经把本地分支push到origin，则应该先删除，否则会拒绝push
		git push origin --delete myFeature
		git push origin myFeature

### 完成本地合并后希望保存分支历史（git默认会把两条分支合并成一个直线） ###

		git merge --no-ff






























