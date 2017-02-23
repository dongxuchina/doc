### Git 的管理策略

[Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)


### Git 常用命令

##### 把项目变成 Git 可以管理的仓库

```git init```

##### 添加远程库

要关联一个远程库：`git remote add origin git@server-name:path/repo-name.git`

关联后，首次推送：`git push -u origin master`

此后，推送最新修改：`git push origin master` 

如果遇到 `! [rejected]        master -> master (non-fast-forward)` 说明服务端有冲突，需要 git pull 合并到本地 `git pull origin master`

##### 克隆远程库

`git clone git@server-name:path/repo-name.git`

##### 把文件添加到暂存区

```
git add file.txt //添加某个文件
git add . //添加全部修改和新增的文件（不包括删除）
git add -u //tracked的文件修改或删除，会被提交
git add -A //添加所有修改、新增和删除文件

```

##### 把文件提交到本地仓库

``` git commit -m "file.txt" ```

##### 查看本地仓库的状态

```git status```

##### 查看提交的日志

``` git log ```

##### 查看某个文件修改的内容

``` git diff file.txt```

##### 回退到某个版本

```
git reset --hard commit_id //回退到指定commit_id
git reset --hard HEAD^  //回退到上个版本 HEAD指向的是当前版本 上个版本 HEAD^ 
上上个版本 HEAD^^  以此类推
```

##### 撤销工作区内某个文件的修改

``` git checkout -- file.txt ```

##### 撤销暂存区内某个文件的修改

``` git reset HEAD file.txt ```

##### 删除版本库中的文件

```
git rm file.txt
rm file.txt
git commit -m "remove file.txt"

```

#### 解决冲突
Git 用 <<<<<<<，=======，>>>>>>> 标记出不同分支的内容，我们修改后保存再提交

```
$ git add file.txt 
$ git commit -m "conflict fixed"
```


##### 分支管理

查看分支：`git branch`

创建分支：`git branch develop`

切换分支：`git checkout develop`

创建+切换分支：`git checkout -b develop`

合并某分支到当前分支：`git merge develop`

删除分支：`git branch -d develop`

删除远程分支：`git push origin :develop`

推送到远程分支：`git push origin develop`

##### 标签管理

查看标签：`git tag`

创建标签：`git tag v2.0`

针对某个commitId创建标签：`git tag v2.0 6224937`

推送标签到远程仓库：`git push origin v2.0`

推送所有的标签到远程仓库：`git push origin --tags`

删除本地标签：`git tag -d v2.0`

删除远程标签：`git push origin :refs/tags/v2.0`

切换标签：`git checkout v2.0`

克隆标签：`git clone -b v2.0 git@server-name:path/repo-name.git`




