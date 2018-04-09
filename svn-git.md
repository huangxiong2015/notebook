## 关于在svn项目中使用git

svn作为一个优秀源码版本的管理工具，可以适合绝大多数项目。但是因为它的采用中心化管理，不可避免的存在本地代码的备份和版本管理问题。也就是说对于尚未或暂无法提交到Subversion服务器的本地代码来说，存在着被误删除和版本更新无法回退两大情形。
git作为一个分布式版本管理工具，可以很好的解决这个问题。因为它的大多数操作是在本地进行的。这里要说的是git是如何做到既可以管理好本地代码又可以与已有的SVN中心库进行同步的。

#### 第一步 将svn上的仓库clone到本地
用vendor-web来做个测试，因为vendor-web里面有2600多个commit，如果全部clone下来会很慢，我们只取其中最后的100个commit，
  ```

    git svn clone -r2500:HEAD https://xxxx:28443/svn/ictrade-vendor/trunk/vendor-web/
  ```
从第2500个commit开始clone仓库，这里clone下来的仓库就是master分支。

#### 第二步 拉取vendor-web的分支
毕竟是svn，拉分支的方式比较繁琐，远没有git好用。现在我们拉取branches/1.4.1.patch这个分支,首先需要进入本地到vendor-web目录

  ```

    cd vendor-web
    git config --add svn-remote.1.4.1.patch.url https://xxxx:28443/svn/ictrade-vendor/branches/1.4.1.patch/vendor-web/
    git config --add svn-remote.1.4.1.patch.fetch :refs/remotes/git-svn-1.4.1.patch
    git svn fetch 1.4.1.patch
    git checkout -b 1.4.1.patch refs/remotes/git-svn-1.4.1.patch
  ```
这样我们就将svn上面的1.4.1.patch这个分支拉取下来的

#### 第三步 既然主干和分支都拉取下来了，我们就可以为所欲为的使用git啦
首先来试试git checkout
  ```

        git checkout master
    	git checkout 1.4.1.patch
  ```
这种切分支的方式真是切的不要太爽。

再来试试提交代码到svn,假设我们需要在1.4.1.patch上面提交代码，首先切到1.4.1.patch分支
```

  	git checkout 1.4.1.patch refs/remotes/git-svn-1.4.1-03.patch


```
再假设我们修改了index.html这个文件，我们可以用git add index.html 或者git add .将所有的改动的代码添加到git仓库，然后提交到本地分支
```

	  git add index.html
	  // 或者 git add .
	  git commit -m "提交index.html到1.4.1分支"

```
代码提交到本地分支之后就应该push到远程仓库了，因为我们是使用svn，所以push的方式跟git还是有很大的区别
```

	  // 要将本地内容提交到远程svn中，可以先让当前分支和远程svn同步
	  git svn rebase
	  // 如果在git svn rebase时发生代码冲突，需要先手动解决冲突，然后用git add将修改加入索引，然后继续rebase
	  git svn rebase --continue
	  // 然后将所有已经提交到1.4.1分支的本地修改提交到svn
	  git svn dcommit

基本的git提交代码流程走通了，接下来就可以随心所欲的使用git来开发了。

```
#### git相关命令

```

	//用远程覆盖本地
	git reset --hard  

	//废弃某个文件
	git checkout -- [file] 

	//#返回到某个节点，不保留修改。
	git reset --hard HASH 

	//#返回到某个节点。保留修改
	git reset --soft HASH 


	//储藏 文件
	git stash

	//显示储藏文件列表
	git stash list

	//应用储藏文件名为 stash@{2}
	git stash apply stash@{2}

	//删除暂存文件名xx
	git stash drop xx

	//回到初始位置
	git stash apply --index

	//取消暂存xx文件的apply
	git stash show -p stash@{0} | git apply -R


```

