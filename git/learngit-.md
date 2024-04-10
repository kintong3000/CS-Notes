#### 常用命令

```shell
git init  //在文件夹目录下执行，通过git init命令把这个目录变成Git可以管理的仓库
git add	<file>	//把文件添加到版本库
git add .		//添加当前目录下的所有文件到暂存区
git commit -m "注释"		//把文件提交到仓库
git status		//仓库当前的状态
git diff <file>		//查看修改内容
git log 		//查看提交历史，其中第一列很长的为commit id（版本号）
git reflog		//查看命令历史
git reset --hard HEAD^		//回退上一版本，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100，也可以写commit id（版本号）不用写全工作区和暂存区
git checkout -- <file>		//丢弃工作区的修改，让这个文件回到最近一次git commit或git add时的状态，详情看撤销修改章节，用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
git reset HEAD <file>		//把暂存区的修改撤销掉
git restore <file>		//新版，==git checkout -- <file>
git restore --staged <file>		//新版，==git reset HEAD <file>
git rm <file>		//删除工作区文件并从版本库中删除该文件
rm -rf .git		//删除本地库
#库
git remote		//查看远程库的信息
git remote add origin git@server-name:path/repo-name.git		//关联远程库
git remote rm <name>		//删除远程库关联
git clone git@server-name:path/repo-name.git`		//克隆远程库

#分支管理
git branch		//查看分支
git branch <name>		//创建分支
git checkout <name>或者git switch <name>		//切换分支
git checkout -b <name>或者git switch -c <name>		//创建+切换分支
git merge <name>		//合并某分支到当前分支 先切换到main，再合并develop分支
git branch -d <name>		//删除分支
git log --graph		//有分支图
git merge --no-ff -m "commit描述" <name>		//在merge时生成一个新的commit
git stash		//缓存分支修改
git stash pop		//恢复分支修改
git cherry-pick <commit id>
git branch -D <name>		//强制删除分支，-d为普通
git push -u origin <Branches>		//推送某分支的所有内容，-u表示建立链接，之后可以直接写成git push
git pull		//拉取分支内容
git branch --set-upstream branch-name origin/branch-name		//建立本地分支和远程分支的关联

git checkout -b my-test  //在当前分支下创建my-test的本地分支分支
git push origin my-test  //将my-test分支推送到远程
git branch --set-upstream-to=origin/my-test //将本地分支my-test关联到远程分支my-test上   
git branch -a //查看远程分支 
git checkout -b dev origin/dev //用于在本地创建一个名为 dev 的新分支，并将其设置为跟踪（track）远程仓库的 origin/dev 分支。
```

#### 版本回溯

##### 工作区和暂存区

**工作区（Working Directory）**

就是在电脑里能看到的目录，比如learngit文件夹就是一个工作区



**版本库（Repository）**

工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为`stage`（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

![git-repo](learngit.assets/0.jpeg)

我们把文件往Git版本库里添加的时候，是分两步执行的：

第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。



##### 管理修改

Git跟踪并管理的是修改，而非文件。如果不用`git add`到暂存区，那就不会加入到`commit`中。



##### 撤销修改

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。



用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区



场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

##### 删除文件

git rm <file> 

小提示：先手动删除文件，然后使用git rm <file>和git add<file>效果是一样的。



#### 远程仓库

##### 添加远程库

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；关联一个远程库时必须给远程库指定一个名字，`origin`是默认习惯命名；



关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

##### 删除远程库

使用前，建议先用`git remote -v`查看远程库信息

然后，根据名字删除，可以用`git remote rm <name>`命令

此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到GitHub，在后台页面找到删除按钮再删除。

##### 从远程库克隆

远程库已经准备好了，可以用命令`git clone`克隆一个本地库：`git clone git@server-name:path/repo-name.git`；

注意：从远程库clone时，默认情况下，只能看到本地的`master`分支。

#### 分支管理

##### 创建与合并分支

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即`master`分支。`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。

一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

```ascii
       						HEAD
                    │
                    │
                    ▼
                 master
                    │
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘
```

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

```ascii
                 master
                    │
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘
                    ▲
                    │
                    │
                   dev
                    ▲
                    │
                    │
                  HEAD
```

从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

```ascii
                 master
                    │
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘    └───┘
                             ▲
                             │
                             │
                            dev
                             ▲
                             │
                             │
                           HEAD
```

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。

```ascii
                           HEAD
                             │
                             │
                             ▼
                          master
                             │
                             │
                             ▼
┌───┐    ┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘    └───┘
                             ▲
                             │
                             │
                            dev
```

此时为`Fast forward`模式合并



因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。



查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

##### 解决冲突

`master`分支和`feature1`分支各自都分别有新的提交，变成了这样：

```ascii
                            HEAD
                              │
                              │
                              ▼
                           master
                              │
                              │
                              ▼
                            ┌───┐
                         ┌─▶│   │
┌───┐    ┌───┐    ┌───┐  │  └───┘
│   │───▶│   │───▶│   │──┤
└───┘    └───┘    └───┘  │  ┌───┐
                         └─▶│   │
                            └───┘
                              ▲
                              │
                              │
                          feature1
```

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突



当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

现在，`master`分支和`feature1`分支变成了下图所示：

```ascii
                                     HEAD
                                       │
                                       │
                                       ▼
                                    master
                                       │
                                       │
                                       ▼
                            ┌───┐    ┌───┐
                         ┌─▶│   │───▶│   │
┌───┐    ┌───┐    ┌───┐  │  └───┘    └───┘
│   │───▶│   │───▶│   │──┤             ▲
└───┘    └───┘    └───┘  │  ┌───┐      │
                         └─▶│   │──────┘
                            └───┘
                              ▲
                              │
                              │
                          feature1
```

用`git log --graph`命令可以看到分支合并图。



##### 分支管理策略

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。



```
$ git merge --no-ff -m "merge with no-ff" dev
```

因为本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。





![git-br-on-master](learngit.assets/0-20230108165636243.png)

![git-no-ff-mode](learngit.assets/0.png)





在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

所以，团队合作的分支看起来就像这样：

![git-br-policy](learngit.assets/0-20230108170002390.png)

##### 缓存分支修改



用`git stash list`命令看存储的工作现场：

```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

恢复有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了：



恢复指定的stash，用命令：

```
$ git stash apply stash@{0}
```



总结：

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场；

##### 复制提交

Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：

在master分支上修复的bug，想要合并到当前dev分支，可以用`git cherry-pick <commit>`命令，把bug提交的修改“复制”到当前分支，避免重复劳动。



##### Feature分支

每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。





#### 多人协作

多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；

2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；

   `git pull`的前提是本地有对应的分支且与远程对应的分支链接；或者git push origin/分支名

3. 如果合并有冲突，则解决冲突，并在本地提交；

4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！



- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；



#### 标签

Git的标签就是指向某个commit的指针

- 命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；
- 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；
- 用命令`git show <tagname>`可以看到说明文字：
- 命令`git tag`可以查看所有标签。
- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。