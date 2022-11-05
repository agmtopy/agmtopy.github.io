---
title: git workflow的几种最佳实践方式
date: 2022-10-29 10:54:10
categories: 杂记
tags:
  - git
---

# git workflow的几种最佳实践方式

## 引言

现有的版本控制系统大多选择使用Git来进行管理/协作;不同的团队有会根据各自的情况选择不同的协助方式,常见的<B>git workflow</B>主要分为三种:

- Git flow
- Github flow
- Gitlab flow
 常见的分支模式也可以大致划分为两种:
 - 主干分支模式
 - 特性分支模式

下面就先介绍特性分支模式下的三种 Git WorkFlow特点与优劣;

## Git flow

> <B>Git flow</B>最早是由Vincent Driessen在2010年左右提出来的<B>[《一个成功的 Git 分支模型》](https://www.ruanyifeng.com/blog/2015/12/git-workflow.html)</B>一文中介绍他自己的分支管理模式;主要核心思想就是会存在两类分支:

- 长期分支:
    - <B>master</B>
    - <B>develop</B>

- 临时分支:
    - <B>hotfixs branch</B>
    - <B>release branch</B>
    - <B>feature branch</B>


长期分支是一直存在的,临时分支只是会存在与一个迭代或一次hotfix的过程中,他们之间的关系如下所示:

![git flow ](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/git_flow.png?raw=true)

也可以看下面这个简化的git flow流程
![git flow ](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/git-flow%E7%AE%80%E5%8C%96%E6%B5%81%E7%A8%8B.png?raw=true)


可以从上面两个图中看到开发者其实是站着<B>develop branch</B>一侧的,feature branch分支通常存在于本地,开发完成后执行<B>merge request(MR)</B>合并到<B>develop branch</B>(通常使用的是git merge --no--ff feature-branch的方式);同时发布分支(release branch)是从develop branch上切出来的,测试通过后mr到master中;

git flow中进行code review的判定点:
- 尝试从私有分支合并代码到公有分支(develop, release/*, hotfix/*, master)中时

例如从feature branch -> develop branch,hotfixs branch -> master/develop时是需要进行code review;
从develop -> master时是不需要进行code review的;


### 优点

- 严谨的合并流程
所有进入公共分支的代码都需要进行审核,确保代码问题;这样的合并流程适用于已有的成熟项目开发流程,可以尽量在前置协作过程中发现避免问题;


- 适用于开源项目(曾经)
各个贡献者都是在各自的repository(存储库)中工作,需要提出PR经过审核后才能提交代码到主库中


### 缺点

- 繁琐
在开发过程中,会持续维护两个长期分支<B>develop</B>和<B>master</B>,并且这两个分支的实际含义还有所重合(develop-拥有不稳定的全部代码的分支/master-拥有全部稳定代码的分支);

- 持续交付困难
所有的功能开发必须是在一个周期或多个周期内完成,造成master中的代码不是最新的,因此很难进行CD;也有基于git flow开进行CD的方案,但是有点舍本求末的感觉在里面了,CI/CD应该是在'每个人每天都致力于master上工作'的前提



### 小结

git flow的开发模式过于繁琐来保证较高的代码质量,需要去维护多个长期分支;繁琐也是相对于下面两种模式而言的;git flow的作者目前也是建议大家基于团队/项目来选择和更简单的GitHub flow;





## GitHub flow

GitHub flow最早是由GitHub的技术经理scott chacon提出来的[GitHub Flow](http://scottchacon.com/2011/08/31/github-flow.html),是基于Github内部使用Git工作的一种流程;

> GitHub flow主要有以下几个特征:
    - master branch中的代码是在任何时候都是可以进行部署的
    - 功能分支名称需要用描述功能特性来进行命名的
    - 即时将本地代码push/pull到服务器上
    - 使用PR来进行协作(反馈/帮助/合并)
    - 完成master合并后立即部署


![github flow](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/github%20flow.jpg?raw=true)

这是一个简易的GitHub flow的流程,可以看到所有去请求在进行审核之后都会被合并到Master分支中;
在实践中GitHub flow是一个一直向前的流程,对于master分支几乎不会进行回滚操作(通过新的提交抵消错误需要进行回滚的合并)


### 优点

- 分支模型简单
这个分支模型简单是相对于Git flow来说的,只用维护一个长期分支<B>master</B>,利于后续的CI/CD

- PR
这个是GitHub flow的一个创新,PR不止是合并代码而是一种协作方式,可以进行评论/审查/帮助,这个是跨地域进行大规模协作的基础,改变了以前通过邮件的方式;

- 可以最大限度减少未发布代码的数量
master branch 在合并代码后就会进入持续交付阶段,这样会最大限度的降低未发布代码的数量



### 缺点

- 无法应对版本的延迟发布
在实际开发工作中,持续交付不一定能实现,毕竟大多数技术开发还是以业务为导向的;

- 无法处理多版本交付
开发环境/测试环境/预发环境甚至生产多版本部署的场景

### 小结

GitHub flow在简化Git flow的基础上支持开源软件的开发模式,但是自身也有一些问题.


## GitLab flow

GitLab flow是由极狐公司提出在<B>[GitLab Flow介绍](https://docs.gitlab.cn/jh/topics/gitlab_flow.html)</B>一文实践的分支管理方式;

GitLab flow的设计遵寻两个原则:
- 单一主分支
> 单一主分支原则与GitHub Flow所保留的Master分支一致

- 上游优先
> 上游优先原则指的是只存在一个主分支master,它是所有其他分支的<B>上游</B>.只有上游分支采纳的代码变化,才能应用到其他分支。对于<B>持续发布</B>的项目，它建议在master分支以外，再建立不同的环境分支。比如,<B>开发环境</B>的分支是master,<B>预发环境</B>的分支是pre-* ,<B>生产环境</B>的分支是pro-*;

![上游优先流程](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/GitLab%20flow%201.png?raw=true)

![生产分支的切出](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/GitLab%20flow%202.png?raw=true)



从上述的流程中可以看到GitLab flow在分支上选择是多分支的管理方式,但是是一种基于<B>上游优先</B>策略下的多分支管理方式,并不是像Git flow那样同时维护多个长期分支,对于后续的发布分支流程,GitLab采用的也是从Master branch中切分支或者打tag的方式:

![GitLab 发布分支](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/GitLab%20flow%203.jpg?raw=true)

在GitLab的实践中,通常<B>Master branch</B>都是受保护的,这样大部分开发者不能对其进行直接修改;
其中在他们的实践中也认可践行(PR/MR)的协助方式,分支的命名也是采用的功能命名的方式,合并后立即删除,以便其他人重新开始这个功能的议题;issues是工作的开始,MR是工作的结束;

### 优点

- 支持多版本部署
支持pre-* /pro-*的多分支部署方式

- 可以支持延迟发布
开发分支和发布分支可以并行


## 小结

由于GitLab flow出现的时间比git flow/gitlab flow都要晚一些,因此吸收了这两种风格的特点(支持多分支/PR模式);
大多数开发模式都是让代码审查通过后直接进入Master branch,因为这样可以尽早的解决冲突;


以上三种代码分支的管理方式都是基于<B>功能开发</B>(先有需求驱动的开发模式),并不是GitLab flow就一定比Git flow要更好,只有更适合的,下面介绍三种简单的区分方式仅供参考:


- 工作中会使用到多个版本 -> Git flow
> 如果代码库在工作中有多个版本(即典型的软件产品,如操作系统、Office 软件包、自定义应用程序等);可以使用git-flow,主要原因是在开发下一个版本的同时,需要在生产中持续支持以前的版本,并且有一个较长的迭代周期;

- 工作中只会使用到一个版本 -> GitHub flow
> 如果代码库始终只有一个生产版本（即网站、Web 服务等），可以使用 github flow。主要原因是您不需要为开发人员复杂的事情。一旦开发人员完成一项功能或完成错误修复，它就会立即升级为生产版本。


- 生产中的单一版本但非常复杂的软件 -> Gitlab-flow

> 在商业大型软件或者是以提供服务的项目上，在生产中可能需要在您的分支和主分支之间来回部署,并且在不同版本都需要进行CI/CD。推荐就使用Gitlab-flow




## 基于主干的开发模式

在基于主干的开发模式中,所有开发人员都在一个开放的分支上进行工作。一般是使用master分支.他们直接向Master提交代码并运行;开发人员会创建短暂的功能分支。一旦他们分支上的代码编译并通过所有测试，他们就会直接将其合并到master. 确保开发是真正连续的，并防止开发人员创建难以解决的合并冲突;

![基于主干开发流程](https://github.com/agmtopy/noteBook/blob/master/png/%E6%9D%82%E8%AE%B0/git_flow/%E5%9F%BA%E4%BA%8E%E4%B8%BB%E5%B9%B2%E7%9A%84%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B.png?raw=true)


可以看到<B>基于主干的开发流程</B>非常的简单,任何人都可以直接向Master branch 合并代码,能够做到快速交付/迭代;缺点是没有人来进行功能性code review,只能进行完整的源代码检查(这是一种灾难😰);

基于主干的开发模式适用于:<B>项目开始的早期</B>、<B>拥有的都是老手</B>(😖)或者是<B>基础架构强/持续集成工具集成度高/TDD和自动化测试覆盖完善场景</B>下;



其实还有其他的开发模式例如<B>集中式</B>、<B>Forking</B>等工作流程,有兴趣的可以了解一下;


## 使用Git的几个约定


在使用Git过程中,其实每个团队的风格不同,都有一些比较好的值得借鉴的地方,以下可能是我自己的一个使用习惯

1. <B>使用rebase -i整理需要提交的commit</B>
每次大功能向master/develop等公开分支提交时,会首先将同一需求的多个commit压缩合并成为一个有具体含义的commit后在发起PR;这样做的原因是基于我认为提交的MR应该是一个完整的功能/补丁/操作的log,可以让之后进行阅读的人知道这一行代码是为那个需求/修复而写的,而不应该是一个需求中的一个小点添加的;第二点是便于回滚/Cherry-pick,一个commit的操作要比多个commit的操作更简单;

2. <B>基于最新的共享分支进行MR</B>
这一点是基于Code review来说的,基于过时的分支合并到共享分支中是会夹杂大量代码差异,不利于code review的进行;对于rebase的使用,坚持一个原则<B>私有分支操作使用rebase,共享分支操作使用merge</B>

3. <B>使用stash</B>
暂存区配合分支切换可以较好的完成,工作区的分支切换/代码存储合并的动作


## 问题

- Code Review如何让每一个人参与其中?

- CI耗时过程问题以及是否需要进行CI?
 > 需要,非常需要;gradle cache;

- 代码文件冲突/合并的问题?
 > 产生的原因是与长期分支脱节、需求划分的不合理、没有及时的协作解决冲突等等


## 参考资料

[GitLab Flow 介绍](https://docs.gitlab.cn/jh/topics/gitlab_flow.html)
[Git 工作流程](https://www.ruanyifeng.com/blog/2015/12/git-workflow.html)
[一个成功的Git分支模式](https://nvie.com/posts/a-successful-git-branching-model/)
[gitLab Flow的11条建议](http://dockone.io/article/2350)
[atlassian Bitbucket](https://www.atlassian.com/git/tutorials/comparing-workflows)
[what-is-git-workflow](https://about.gitlab.com/topics/version-control/what-is-git-workflow/)
[trunk based developmentgit flow](https://www.toptal.com/software/trunk-based-development-git-flow)
[State of CI/CD and the omnipresent git flow](https://medium.com/burdaforward/state-of-ci-cd-and-the-dreaded-git-flow-fce92d04fb07)