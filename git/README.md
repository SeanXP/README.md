#git学习笔记

###创建git目录.

	$ git init

####查看git版本
	
	$ git --version
 
####配置用户信息
		
	$ git config --global user.name "Sean Guo"
	$ git config --global user.email "iseanxp@gmail.com"

####开启git输出颜色现实
	
	$ git config --global color.ui true



###添加某文件(提交commit前必须执行添加命令)
	$ git add test.txt
快速标记添加已跟踪的文件
	
	$ git add -u
选择性添加

	$ git add -i

-------

###提交

	$ git commit -m "some need to say..."
允许空白提交(--allow-empty选项)

	$ git commit --allow-empty
	
修改刚才提交的注释信息：

	$ git commit --amend -m ”YOUR-NEW-COMMIT-MESSAGE”

假如已经将代码提交（git commit）推送（git push）到了远程分支，那么你需要通过下面的命令强制推送这次的代码提交。
	
	$ git push <remote> <branch> --force

------------------------------------------

###查看git提交日志
	$ git log
查看详细日志(显示文件变更统计日志)
	
	$ git log --stat
查看精简日志(一个提交只显示一行)

	$ git log --pretty=oneline
查看提交跟踪链(指向commit的parent)

	$ git log --graph
------------------------------------------
###查看当前git状态

	$ git status
查看精简git状态

	$ git status -s

执行git status命令, 会扫描工作区的改动情况。     
先依据.git/index文件(用于跟踪工作区文件)记录的时间戳, 长度等信息,
(利用时间戳使得扫描改动变得更高效,如果时间戳相同, 就不用查看内容,
 如果时间戳不同, 再查看内容, 如果内容没变, 则更新时间戳, 否则显示消息)。

.git/index是暂存区的目录树,记录文件名和文件状态信息(时间戳,长度等),
而真正文件内容存储于git对象库.git/objects目录中.文件索引建立了文件与对象库的连接.

------------------------------------------

###查看当前git目录与最新提交的区别	
	$ git diff

git目录下有三个版本,  
    
1. 一个是工作区, 是最新的版本
2. 一个是等待提交的暂存区(stage), 是最新的git add的版本      
3. 一个是已提交中的版本, 是最新的commit的版本.      

git diff默认对比[ 工作区与stage暂存区 ], git diff HEAD将[ 工作区与HEAD版本 ] (当前版本分支)对比.
      

git diff --cached , git diff --staged是将[ stage暂存区与HEAD版本库 ] 对比
	
	$ git diff --cached
	$ git diff --staged

------------------------------------------
###目录树浏览
显示HEAD中的目录树, -l 显示文件大小.

	$ git ls-tree -l HEAD

显示stage暂存区目录树, -s 显示详细内容

	$ git ls-files 
	$ git ls-files -s

------------------------------------------
###查看ID(40位哈希值).

查看log的ID输出. 输出commit ID, tree ID, parent ID.

	$ git log -l --pretty=raw

查看ID对应的文件类型

	$ git cat-file -t [ID]

查看ID对应的文件内容
	
	$ git cat-file -p [ID]

查看id的内容
	
	$ git show [ID]
------------------------------------------
###查看master分支切换历史
	$ git reflog show master
------------------------------------------

###git reset [-q] [<commit>] [--] <paths>...
如果你往暂存区staging area中加入了一些错误的文件，但是还没有提交代码。你可以使用一条简单的命令就可以撤销。如果只需要移除一个文件，那么请输入：

	$ git reset <文件名>
或者如果你想从暂存区移除所有没有提交的修改：

	$ git reset

可以指定路径, 加入'--'避免路径和引用冲突;     
<commit>可以是引用或者ID, 制定哪一次提交的版本库. 不写默认为HEAD.
将已提交状态<commit>下的文件<paths>替换掉暂存区的文件.     
eg. 
	
		$ git reset HEAD <paths>
相当于去取消之前git add <paths>所改变的暂存区.

###git reset [--soft | --mixed | --hard | --merge | --keep] [-q] [<commit>]
重置引用.    
可以对暂存区或工作区进行重置.     

1. 替换引用的指向, 即将引用(当前引用是HEAD)中包含的commit的ID替换, 改变引用所指向的commit.
2. 替换暂存区, 将暂存区index的目录树内容 和 引用指向的内容相同.
3. 替换工作去, 将工作区的内容变得和暂存区内容一致.

其中,     
--hard 选项, 执行1,2,3操作.     
--soft 执行1操作,只改变引用HEAD指向.     
--mixed(默认情况),执行操作1,2, 即更改HEAD引用的指向以及暂存区。     

eg.

	$ git reset (相当于--mixed选项,但是没有提供<commit>,故HEAD的指向没有改变, 只是将暂存区改于HEAD一致)
	$ git reset HEAD (效果同上)
	$ git reset -- filename (将HEAD中的filename覆盖暂存区, 即取消了git add filename的影响,相当于反操作)

	$ git reset --soft HEAD^ (HEAD^表示HEAD的parent commit, 即上一次提交;即只改变了HEAD的指向,向上回溯)     
	(用于对刚才的提交不满意, 不改动暂存区与工作区, 而重置HEAD以重新提交.)


重置git reset的默认值是HEAD, 检出git checkout 的默认值是暂存区.      
因此, 重置一般用于重置暂存区(除非使用hard参数,否则不会覆盖工作区),
检出命令主要用于覆盖工作区。
	
	
------------------------------------------
###git checkout命令

	git checkout [-q] [<commit>] [--] <paths>...
从制定的提交<commit>中,检出<paths>路径文件,覆盖工作区与暂存区.(默认情况,默认值为暂存区,则只覆盖工作区)
	
	git checkout [<branch>]
切换HEAD引用到分支 <branch>. 1.更新HEAD引用指向分支branch; 2.使用branch覆盖暂存区与工作区.

	git checkout [-m] [ [-b | --orphan] <new_branch> ] [<start_point>]
创建和切换到新的分支 <new_branch>, 且新分支从<star_point>制定的提交创建.

eg:
	
	$ git checkout branch (切换分支,且更新暂存区与工作区)
	$ git checkout (汇总显示工作区, 暂存区, HEAD的差异)
	$ git checkout -- filename (使用暂存区(默认)的filename覆盖工作区)
	$ git checkout branch -- filename (使用branch分支中的filename直接覆盖暂存区与工作区的文件)
	$ git checkout . (危险! 直接使用暂存区覆盖工作区的所有文件, 即取消所有工作区的修改)



	$ git checkout -b bug1  (创建一个新分支, 命名为bug1)
	$ git diff master bug1  (对比两个分支的区别)
	$ git merge bug1        (将当前分支与bug1分支合并)
	$ git branch -d bug1    (删除分支bug1)

------------------------------------------
保存当前的工作进度.(分别对工作区/暂存区进行保存)
	
	$git stash

显示进度列表.

	$git stash list

不使用参数, 则默认恢复最新保存的工作进度。     
否则回复对应<stash>的进度,且恢复完成后在stash list中删除相应记录.
[--index]表示除了恢复工作区,还将尝试恢复暂存区.
	
	$ git stash pop [--index] [<stash>]

与git stash pop相同的恢复,但不在stash list中删除记录.
	
	$ git stash apply [--index] [<stash>]

完整版本命令.

	$ git stash [save [--patch] [-k | --[no-]keep-index ] [-q | --quiet] [<message>]

保存当前进度并添加信息

	$ git stash save "message..."

删除存储进度. 不制定<stash>则默认删除最新进度.
	
	$ git stash drop [<stash>]

删除所有存储进度.（慎用）

	$ git stash clear

基于进度创建分支.

	$ git stash branch <branchname> <stash>

------------------------------------------
###Head^

HEAD表示版本库中最近的一次提交.    
HEAD^表示上一次提交.     
HEAD^^表示HEAD^的父提交.     

但是如果一个提交有多个父提交呢?    
HEAD^1表示HEAD的多个父提交中的第一个父提交.    
HEAD^2表示HEAD的多个父提交中的第二个父提交.    
HEAD^^2表示HEAD^的多个父提交中的第二个父提交.     

HEAD~5 相当于HEAD^^^^^

查看提交对应的树对象
HEAD^{tree}

查看提交的某具体文件
HEAD:path/to/file

上面的HEAD都可以替换为其他引用或者具体的ID.

------------------------------------------
###创建里程碑 tag
	$ git tag -m "this is my first tag" first_tag

查看当前里程碑(显示创建的里程碑,或由最近里程碑为基础版本叠加的里程碑)
	
	$ git describe

------------------------------------------
###删除 rm
	$ git rm filename

恢复
	
	$ git checkout HEAD~1 -- somefile

移动文件

	$ git mv welcome.txt README

强制撤销提交,覆盖工作区与暂存区

	$ git reset --hard HEAD^
------------------------------------------
###文件忽略 ignore 
在目录下, 添加.gitignore文件.
在里面可以添加要忽略的文件, 如*.o, *.h, *.out
甚至可以忽略.gitignore本身.    
.gitignore的作用范围是本目录及其子目录.

可以使用下面命令查看被忽略的文件.
	
	$ git status --ignored 

如果要在屏蔽*.h的情况下添加一个具体的hello.h

	$ git add -f hello.h
必须制定名称使用git add -f. -f参数不能少.

.gitignore的示例:
		
	#这是注释行
	*.a         # 屏蔽后缀为a的文件     
	!lib.a      # 但是lib.a不要被忽略      
	/Todo       # 只忽略此目录下的Tode文件, 不忽略子目录下的Tode文件
	build/      # 忽略所有build/目录下的文件        
	doc/*.txt   # 忽略类似文件doc/notes.txt, 但不忽略doc/server/arch.txt.       

注意: 上面只是示例, 不能按照上面的写法, 要过滤的文件写在一行, 后面不能有任何东西, 包括空格.
eg. 

	#屏蔽tags文件,只有4个字母, 后面不能有空格, 否则过滤失败
	tags
------------------------------------------
###文件归档
基于最新提交建立归档文件latest.zip;
	
	$ git archive -o latest.zip HEAD

只将目录src/和doc/建立到归档partial.tar中
	
	$ git archive -o partial.tar HEAD src doc

基于里程碑v1.0建立归档, 并为归档中的文件添加目录前缀1.0

	$ git archive --format=tar --prefix=1.0/ v1.0 | gzip > foo-1.0.tar.gz

------------------------------------------
###克隆
	$ git clone <repository> <directory>
将<repository>指向的版本库创建一份克隆至<directory>目录.

	$ git clone --bare <repository> <directory.git>
同上, 克隆至制定目录, 但只克隆.git, 不包含工作区, 称之为裸版本库;
一般约定裸版本库的目录名以.git为后缀, 如project1.git;

	$ git clone --mirror <repository> <directory.git>
同第二个, 克隆裸版本库.
与第二个相比, 此用法对上游版本进行了注册, 可以与上游版本库保持同步.

	$ git push
	$ git pull
------------------------------------------

1. 直接使用git clone <repository> <directory> 创建一个备份库。
因为备份的库对上游库进行了注册, 所以上游库有什么更改, 在备份库可以通过git pull实现同步.
但是,上游库并不知道克隆库的信息, 所以无法git push.
	
		git clone <repository> <directory>
	

2. 使用git clone --bare创建一个裸版本库, 类似1, 只能在克隆库中git pull而无法在原始库中git push,
但裸版本库占的空间更小.
如果制定push路径的话,在原始库还是可以push的.

		$ git push /path/to/repos/demo.git

------------------------------------------
###git object文件清理

查看没有被版本库任何引用关联的松散对象
	
	$ git fsck

彻底清除没有未关联的松散对象
	
	$ git prune
------------------------------------------
###git cherry-pick
	$ git cherry-pick C2 (在当前提交下, 重复一遍提交C2的操作.)



###git rebase

	$ git rebase C3 master (将分支master转移到特定提交C3上去, 分支的转移)
	
	$ git rebase C1 (从当前提交到C1提交之间的所有commit, 都可以被编辑, 可以被修改提交顺序)


------
github使用方法

1. 在github上创建项目
2. 在本地创建目录
3. 在本地目录执行
	
	`git init`
	
	`git remote add origin git@github.com:xxx/xxx.git`
	
	`git pull origin master`
	
	`git push origin master`
	
	