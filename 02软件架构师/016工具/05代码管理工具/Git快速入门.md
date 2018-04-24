由于GIT刚刚开始使用不久，经常会在Merge时出现没有change-id的情况，在结合gerrit使用时，经常出现不能提交的情形，使得自己很困扰。最近有次熬夜加班，在代码完成后，由于多人在很短时间内提交多次，造成提交不上去的情况，当时那个慌啊。还好有给力的大神帮忙处理，突然觉得有必要好好学学Git了，不能仅仅满足于图形化工具的使用。

# 基础概念
**工作区、版本库、缓存库的关系和区别，如下图所示**
![](https://images2015.cnblogs.com/blog/636325/201602/636325-20160222181728770-959132880.png)

* 工作区：左侧的工作区就是我们日常编辑的部分
* 暂存区：使用git add XXX后的部分
* 版本库：使用commit后的部分
* HEAD：当前版本指针

**Git中Tree, Blob, Commit, Tag**

* Blob: 就是一块内存区域，其中内容可以是文本，源码或者图片
* Tree: 类似文件系统中的目录，可以指向blob或者其他的树，就像目录可以包含文件和子目录一样
* Commit: 包含提交者的信息（姓名，Email），指向当前提交下所属的tree的指针，指向之前提交（父提交）的指针
* Tag: 包含指向任意commit的指针，便于记录和使用指定的tree，而不用使用哈希值。

# 常见命令
|git命令|解释|
|:--|:--|
|git config --global  --list | 查看本地配置环境变量|
|git config --global user.name | Git相关环境变量配置, user.email, --local|
| git status| 查看当前状态|
| git diff| |
| git checkout <branch>| 切换到指定branch|
|git checkout –b new_branch |出并创建分支，git pull origin xxx 本地分支与远程分支相关联 |
|git branch –r | 查看远程分支|
|	git branch --set-upstream-to origin/devtest devtest|关联到指定分支|
| git commit –m 'XXX'	| 提交并添加评论，需要注意的是提交什么的哈希码，是通过SHA1算法获得的160bit哈希值，**在分布式版本控制系统中需要使用SHA1来唯一标识，而不是顺序ID。**|
|git commit --amend | 对刚刚的提交进行修补，不会产生新的提交。有时，在merge操作后，在gerrit出现不能提交的情况，可以通过该命令，为merge commit产生一条changeID。其原理其实是组合了两条git命令，如下所示:|
| git add -i| 进入交互式界面添加/修改/删除文件到暂存区，**git add .快速添加所有文件**|
|git merge |git merge origin/release, git merge XXX，**分为两步，先将远程release内容merge到本地branch,之后再merge到release，比较稳妥**|
|git merge --no-ff|None fast-forward，不快速向前|
|git rebase | |
|git rebase –i|合并提交，将**pick改为fixup**|
| git clean –nd |	查看那些文件和目录会被删除 |
|git clean –fd | 强制删除多余的目录和文件|
| git reset --soft <commit>| 包含1个步骤：a.替换引用的指向 |
|git reset --mixed <commit>，默认方式 | 包含2个步骤：a.替换引用的指向；b.替换暂存区|
| git reset --hard some_commit_hashcode| 包含3个步骤：a.替换引用的指向；b.替换暂存区；c.替换工作区|
|git reset --soft HEAD^| 工作区和暂存区不变，但引用向前回退一次|
|git stash| 保存当前进度，会分别对暂存区和工作区的状态进行保存，完整版本为git stash save "XXX"|
| git log -i|查看指定条目数日志|
|git log --graph | 	以图形化的方式查看日志，在理解分支信息时很有用|
| git clone XXX| 拉代码，可以通过https和ssh等方式|
| git tag| 查看里程碑列表|
| git tag tagname	| 轻量级里程碑，在.git/refs/tags中。其他还是重量级里程碑等内容，包括数字签名|
| git cherry-pick <commit>| 	用于把另一个本地分支的commit修改应用到当前分支|
| **git revert HEAD**| 将HEAD提交反向再提交一次。由于修改历史操作只能是针对自己的版本库，而无法去修改别人的版本库，这时就可能需要使用revert去修正一个错误的历史提交|
|git push| 注意要禁止非快进式推送，理解不深|
| git push origin :branch-name| git **删除远程分支**,冒号前空格不能少，表示把一个空分支push到server,相当于删除|
|git push -u origin release　 |创建远程分支,配合git branch release |
| git gc| 垃圾回收，清理版本库|
| git fetch|获取最新代码信息，但不自动merge |
| git pull| 拉去本地分支关联的远程分支的内容，并merge|
|git remote [-v]| 查看当前本地分支track的远程分支|
| git remote show origin| 显示远程所有的分支信息|
| git remote rename [a] [b]| 重命名远程库|
| git remote  remove [a]| 删除远程仓库|

# 补充知识
Android项目包含近200个Git版本库，因而google公司开发了repo(对git的封装)和gerrit两个工具进行版本库管理，其中gerrit是一种特别的集中式协同模型，通过SSH协议管理Git版本库，并实现一个Web界面的评审工作流。其中，Git不能直接推送到分支，而是推送到特殊的引用refs/for/<branch-name>。

其中困扰我良久的change-id其实不是git中的概念，而是gerrit中的概念，它通过hooks的方式(其实就是面向切面的拦截器，在C语言中一般叫做hooks钩子，位于.git\hooks\文件夹中)为该次提交添加一个change-id，然后就可以被gerrit管理起来了。`**Gerrit提交方式，Git push origin HEAD:refs/for/your_brance...%r=xxx。**`

**Gerrit审核服务器**最初其实是为Android项目开发。Redmine是一款实现需求管理和缺陷跟踪的项目管理软件，可以和Git版本库实现整合，git的提交可以之间关闭redmine上的Bug,同时Git的提交还可以反映出项目成员的工作进度。Redmine中的用户（项目成员）用一个ID做标识，而Git的提交者则用一个包含用户名和邮件地址的字符串，需要一个关联配置。
Git模型图如下所示
![](https://images2015.cnblogs.com/blog/636325/201606/636325-20160601080654149-971010202.png)

Git设置SSH
生成本机`ssh key, ssh-keygen -t rsa -C "xxxxxx@yy.com"`

在git中如果想忽略掉某个文件，不让这个文件提交到版本库中，可以使用修改 .gitignore 文件的方法。这个文件每一行保存了一个匹配的规则例如：
此为注释 – 将被 Git 忽略

	*.a       # 忽略所有 .a 结尾的文件
	!lib.a    # 但 lib.a 除外
	/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
	build/    # 忽略 build/ 目录下的所有文件
	doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt



一个gitignore的问题： [.gitignore不生效问题](http://blog.csdn.net/qinyushuang/article/details/55210286)
原因明白了，是那个文件已经被git管理，ignore无效。。需要delete
在IDEA中，可以安装CVS插件

idea 终端 git命令智能感知
http://blog.csdn.net/qq_28867949/article/details/73012300
查找git默认安装目录`which git`
`/usr/bin/git-shell`


**Tip**
主要供自己工作参考，若有疏漏，望见谅。参考蒋鑫大师的《GIT权威指南》和大塚弘记的GitHub入门与实践，前者非常全面，后者简单有效。


