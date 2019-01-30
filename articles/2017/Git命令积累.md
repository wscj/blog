记录自己使用过的git命令

**仓库**
```
$ git clone <repository> [folder] # 克隆远程仓库
$ git clone -b <branch> <repository> [folder] # 克隆指定分支
$ git remote add <remote> <respository> # 添加远程仓库
$ git remote show <remote> # 查看远程信息
```

**分支**
```
$ git branch <branch> # 新增分支
$ git branch # 查看本地分支
$ git branch -r # 查看远程分支
$ git branch -b <branch> # 新增分支并切换到该新分支
$ git branch -d <branch> # 删除本地分支
$ git branch -D <branch> # 强制删除本地分支
$ git push <remote> :<branch> # 删除远程分支，例：git push origin :dev
$ git pull <remote> <branch> # 获取远程指定分支最新版本
$ git push <remote> <branch> # 推送本地版本到远程指定分支，如果远程不存在同名分支则创建
$ git fetch <remote> # 获取远程所有分支的更新，包括新增加的远程分支
$ git fetch -p # 刷新远程分支列表（当远程分支被他人删除，自己的远程分支列表不会自动刷新）
$ git branch --track <branch> <remote>/<branch> # 新建分支，与指定的远程分支建立追踪关系
$ git checkout -b <branch> # 新建分支并切换到该分支上
```

**版本切换**
```
$ git reset --hard HEAD~n # 往前回滚n个版本
$ git reset --hard <commitID> # 切换到指定 commit id 对应的版本
$ git reset <commitID> #  撤销commit，修改内容仍然存在
```

**删除新增的文件**
```
$ git clean -fd  # 删除未添加进暂存区的新增文件
$ git clean -nd .  # 在使用上面的删除命令之前，可使用该命令查看将会被删除的文件
```

**暂存区**
```
$ git add file_name # 单个文件添加进暂存区
$ git add . # 把所有修改添加进暂存区
$ git reset HEAD # 把暂存区的内容撤销出去
$ git reset HEAD file_name # 把单个文件撤出暂存区
$ git stash # 隐藏暂存区的内容，回到干净的状态
$ git stash pop # 释放隐藏，还原之前的内容
$ git checkout . # 清除暂存区中的所有修改
```

**合并**
```
$ git merge <branch> # 把另一个分支合并到当前分支
$ git reset --hard HEAD # 撤销合并，常用于撤销pull后的合并
```

**标签**
```
$ git tag v1.0.0 # 创建轻量级标签
$ git tag -a v1.0.0 -m 'remark' # 创建含附注的标签
$ git fetch <remote> tag <tag> # 获取远程标签
$ git fetch --tags # 获取所有标签
$ git push --tags # 一次性推送本地的所有标签到远程仓库
$ git tag -d <tag> # 删除本地标签
$ git push --delete <remote> <tag> # 删除远程tag
```

**其他**
```
$ git config -l # 查看本地配置信息
$ ssh-keygen -f ~/.ssh/somebody # 生成公钥密钥，如~/.ssh文件夹不存在则需要先创建
$ git config --global user.name '用户名'
$ git config --global user.email '邮箱'
$ git config --global --unset user.email # 删除已经全局配置的邮箱，这里可以把邮箱替换成其他的属性
$ git config --global gui.encoding utf-8 #  设置编码，解决gitk中文乱码问题
$ git rm --cached <file> # 停止追踪指定文件
$ git blame <file> # 查看指定文件，具体到每行代码最后的修改信息（包括修改人与修改时间）
$ git commit --amend --author="xxx <xxx@xxx.xxx>" --no-edit #  修改最后一次commit的作者信息
$ git cherry-pick <commitID> #  从其他分支提取某个commit合并过来
```
