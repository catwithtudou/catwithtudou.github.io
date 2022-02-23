# Git中的工作流


# Git中的工作流

## 中心式协同工作流

一般过程如下：

1. 从服务器上`git pull origin master`同步代码；
2. 修改完后`git commit`到本地仓库中；
3. 然后`git push origin master`到远程仓库中；

若三步push失败则出现了已经提交的版本与你本地版本不一致，则需要先把远程仓库的代码pull下来，为了避免有merge操作即可以使用`git pull -rebase`将远程仓库上的提交直接合并到本地仓库中。总结就是以下步骤：

1. 先把本地提交的代码放在一边；
2. 然后把远程仓库上的改动下载下来；
3. 然后在本地把之前的改动全部分别commit直至全部成功。

![image-20201230174210056](https://img.zhengyua.cn/img/20201230174217.png)



> 若有冲突则解决冲突，使用`git rebase –continue`，直至解决所有冲突。
>
> ![image-20201230174250144](https://img.zhengyua.cn/img/20201230174250.png)

## 功能分支协同工作流

一般过程如下：

1. 首先使用`git checkout -b new-feather`创建新分支；
2. 然后共同开发这个功能在此分支上进行开发工作；
3. 然后通过`git push -u origin new-feather`把分支push到远程仓库上；
4. 其他就可以通过`git pull -rebase`拿到这个分支的最新代码；
5. 最后通过`Pull Request`做完CodeReveiw后合并到Master分支上。

![image-20201230174558219](https://img.zhengyua.cn/img/20201230174558.png)

## GitFlow协同工作流

**重点在于维护且不影响当前可以发布的代码**。

![image-20201230174644118](https://img.zhengyua.cn/img/20201230174644.png)

- Master 分支。也就是主干分支，用作发布环境，上面的每一次提交都是可以发布的。

- Feature 分支。也就是功能分支，用于开发功能，其对应的是开发环境。

- Developer 分支。是开发分支，一旦功能开发完成，就向 Developer 分支合并，合并完成后，删除功能分支。这个分支对应的是集成测试环境。

- Release 分支。当 Developer 分支测试达到可以发布状态时，开出一个 Release 分支来，然后做发布前的准备工作。这个分支对应的是预发环境。之所以需要这个 Release 分支，是我们的开发可以继续向前，不会因为要发布而被 block 住而不能提交。

  > 一旦 Release 分支上的代码达到可以上线的状态，那么需要把 Release 分支向 Master 分支和 Developer 分支同时合并，以保证代码的一致性。然后再把 Release 分支删除掉。

- Hotfix 分支。是用于处理生产线上代码的 Bug-fix，每个线上代码的 Bug-fix 都需要开一个 Hotfix 分支，完成后，向 Developer 分支和 Master 分支上合并。合并完成后，删除 Hotfix 分支。

## GitHub/Gitlab协同工作流

> GitFlow会因分支太多造成git log混乱的局面，且需要同时维护Master和Developer分支。

### GitFlow

一般过程如下：

1. 将官方库fork到自己的代码仓库；
2. 开发人员在自己的代码仓库进行开发；
3. 自己的代码仓库中需要配置两个远程仓库；
4. 本地建立功能分支，在此分支上做代码开发；
5. 这个功能分支push到自己的代码仓库中；
6. 然后向官方库发起pull request，并做code review；
7. 一旦通过则向官方库进行合并。

### GillabFlow

**重点在于引入不同的环境分支，且包含了预发布和生产分支**。

![image-20201230175402554](https://img.zhengyua.cn/img/20201230175402.png)

![image-20201230175412503](https://img.zhengyua.cn/img/20201230175412.png)

这样也就解决了两个问题：

- 环境和代码分支对应的问题；
- 版本和代码分支对应的问题。

## 本质

软件开发的趋势一般如下：

- 以微服务或者SOA架构的方式。

  使协同工作流变得简单，将各个服务拆解成若干个代码仓库，对于GitFlow过于重，更适合使用GitHubFlow方式。

- 以DevOps为主的开发流程。

  通过CI/CD，不需要维护多个版本，也不需要关注不同的运行环境，GitHubFlow或者功能分支这种方式更适应这种开发流程。

协同工作流的本质，并不是怎么玩好代码仓库的分支策略，而是玩好我们的软件架构和软件开发流程。

与其花时间在 Git 协同工作流上，还不如把时间花在调整软件架构和自动化软件生产和运维流程上来，这才是真正简化协同工作流程的根本。
