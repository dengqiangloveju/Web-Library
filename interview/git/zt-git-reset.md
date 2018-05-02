## git 远程仓库版本回退方法



#### 第一篇：自己的远程分支版本回退的方法



假设你现在的graph 是这个样子的



![](/Users/liujunbin/Documents/project/mygithub/FETopic/interview/assets/git-reset1.png)





正常情况下， 这个点的排序是按照时间来处理的， 对于有直线洁癖的人来说， 我们现在是需要把这根线拉直



1.第一步,查看下面的 log

```nginx
git reflog
```



![](/Users/liujunbin/Documents/project/mygithub/FETopic/interview/assets/git-reset2.png)



2.分析第一张图， 我们发现 我们需要 把远程历史提交记录回退到 add 777 的位置（也就是第一张图的第三个提交）， 可以使用如下命令



```nginx

// 回退到某个节点
git reset --hard 9bd1984

// 强制push 上去

git push -f 
```



**注意：本地分支回滚后，版本将落后远程分支，必须使用强制推送覆盖远程分支，否则无法推送到远程分支**







#### 第二篇：公共远程分支版本回退的问题



看到这里，相信你已经能够回滚远程分支的版本了，那么你也许会问了，回滚公共远程分支和回滚自己的远程分支有区别吗？ 
答案是，当然有区别啦。



> 一个显而易见的问题：如果你回退公共远程分支，把别人的提交给丢掉了怎么办？

下面来分析:

假如你的远程master分支情况是这样的:



> A1–A2–B1

其中A、B分别代表两个人，A1、A2、B1代表各自的提交。并且所有人的本地分支都已经更新到最新版本，和远程分支一致。

这个时候你发现A2这次提交有错误，你用reset回滚远程分支master到A1，那么理想状态是你的队友一拉代码git pull，他们的master分支也回滚了，然而现实却是，你的队友会看到下面的提示：



```nginx
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean12345
```

也就是说，你的队友的分支并没有主动回退，而是比远程分支超前了两次提交，因为远程分支回退了嘛。

(1) 这个时候，你大吼一声：兄弟们，老子回退版本了。如果你的队友都是神之队友，比如: Tony(腾讯CTO)，那么Tony会冷静的使用下面的命令来找出你回退版本后覆盖掉的他的提交，也就是B1那次提交：

```nginx
git reflog1
```

然后冷静的把自己的分支回退到那次提交，并且拉个分支:

```nginx
git checkout tony_branch        //先回到自己的分支  
git reflog                      //接着看看当前的commit id,例如:0bbbbb    
git reset --hard B1             //回到被覆盖的那次提交B1
git checkout -b tony_backup     //拉个分支，用于保存之前因为回退版本被覆盖掉的提交B1
git checkout tony_branch        //拉完分支，迅速回到自己分支
git reset --hard 0bbbbbb        //马上回到自己分支的最前端123456
```

通过上面一通敲，Tony暂时舒了一口气，还好，B1那次提交找回来了,这时tony_backup分支最新的一次提交就是B1，接着Tony要把自己的本地master分支和远程master分支保持一致：

```nginx
git reset --hard origin/master1
```

执行了上面这条命令后，Tony的master分支才真正的回滚了,也就是说你的回滚操作才能对Tony生效，这个时候Tony的本地maser是这样的：

> A1

接着Tony要再次合并那个被丢掉的B1提交：

```nginx
git checkout master             //切换到master
git merge tony_backup           //再合并一次带有B1的分支到master12
```

好了，Tony终于长舒一口气，这个时候他的master分支是下面这样的：

> A1 – B1

终于把丢掉的B1给找回来了，接着他push一下，你一拉也能同步。

**同理对于所有队友也要这样做，但是如果该队友没有提交被你丢掉，那么他拉完代码git pull之后，只需要强制用远程master覆盖掉本地master就可以了**：

```nginx
git reset --hard origin/master1
```

(2) 然而很不幸的是，现实中，我们经常遇到的都是猪一样的队友，他们一看到下面提示：

```nginx
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean12345
```

就习惯性的git push一下，或者他们直接用的SourceTree这样的图形界面工具，一看到界面上显示的是推送的提示就直接点了推送按钮，卧&槽，你辛辛苦苦回滚的版本就这样轻松的被你猪一样的队友给还原了，所以，只要有一个队友push之后，远程master又变成了：

> A1 – A2 – B1

这就是分布式，每个人都有副本。这个时候你连揍他的心都有了，怎么办呢？你不能指望每个人队友都是git高手，下面我们用另外一种方法来回退版本。



> **注意：博主是在虚拟机中实验的，用于模拟两个人的操作，如果你在一个机器上，用同一个账号在不同的目录下克隆两份代码来实验的话，回退远程分支后，另外一个人是不会看到落后远程分支两次提交的，所以请务必使用虚拟机来模拟A、B两个人的操作**



#### 第三篇： 公共远程分支版本回退的方法



使用git reset回退公共远程分支的版本后，需要其他所有人手动用远程master分支覆盖本地master分支，显然，这不是优雅的回退方法，下面我们使用另个一个命令来回退版本：

```nginx
git revert HEAD                     //撤销最近一次提交
git revert HEAD~1                   //撤销上上次的提交，注意：数字从0开始
git revert 0ffaacc                  //撤销0ffaacc这次提交123
```

git revert 命令意思是撤销某次提交。它会产生一个新的提交，虽然代码回退了，但是版本依然是向前的，所以，当你用revert回退之后，所有人pull之后，他们的代码也自动的回退了。 
但是，要注意以下几点：

> 1. revert 是撤销一次提交，所以后面的commit id是你需要回滚到的版本的前一次提交
> 2. 使用revert HEAD是撤销最近的一次提交，如果你最近一次提交是用revert命令产生的，那么你再执行一次，就相当于撤销了上次的撤销操作，换句话说，你连续执行两次revert HEAD命令，就跟没执行是一样的
> 3. 使用revert HEAD~1 表示撤销最近2次提交，这个数字是从0开始的，如果你之前撤销过产生了commi id，那么也会计算在内的。
> 4. 如果使用 revert 撤销的不是最近一次提交，那么一定会有代码冲突，需要你合并代码，合并代码只需要把当前的代码全部去掉，保留之前版本的代码就可以了.

git revert 命令的好处就是不会丢掉别人的提交，即使你撤销后覆盖了别人的提交，他更新代码后，可以在本地用 reset 向前回滚，找到自己的代码，然后拉一下分支，再回来合并上去就可以找回被你覆盖的提交了。





#### 第四篇：继续扩展，简单粗暴的回滚方法



看到这里也许你已经觉得学会了远程仓库版本回滚方法了，但是实践中总是会遇到很多不按套路来的问题，考虑下面一种情况：

> 如果你们开发中，忽然发现前面很远的地方有一次错误的合并代码，把本来下一次才能发的功能的代码合并到了这一次来了，这个时候全体成员都觉得直接回滚比较快，因为他们都有备份，覆盖了无所谓，这个时候用reset的话对队友的要求比较高，用revert的话呢要大面积的解决冲突，也很麻烦呀，怎么办呢？

这个时候，可以使用简单粗暴的办法，直接从那个错误的提交的前一次拉取一份代码放到其他目录，然后将master代码全部删除，把那份新代码方进去，然后提交，果然简单粗暴啊，虽然这种方法不入流，但是，实践中发现很好使啊



## 总结

远程分支回滚的三种方法：

> 1. 自己的分支回滚直接用reset
> 2. 公共分支回滚用revert
> 3. 错的太远了直接将代码全部删掉，用正确代码替代





![结果线直了！！](/Users/liujunbin/Documents/project/mygithub/FETopic/interview/assets/git-reset3.png)