# Git工作流中常见的三种分支策略：GitFlow、GitHubFlow以及GitLabFlow

【摘要】 介绍Git工作流中常见的三种分支策略：GitFlow、GitHubFlow以及GitLabFlow。

# 前言

版本控制系统是指对软件开发过程中程序代码、配置文件、文档等发生的变更进行管理的系统，它可以帮助团队更好的沟通协作，从而更好的进行交付，常见的版本控制系统分为集中式版本控制系统（如SVN）和分布式版本控制系统（如Git）。

之前拜访一家企业，企业内的开发团队使用Git管理日常开发工作，在开发过程中遇到一个问题：分支策略使用很混乱——最初开发团队从主干分支拉出一条特性分支，但新功能完成后，该特性分支没有合入主干分支，而是作为下次开发的主干分支，重新拉出一条新的特性分支，导致主干分支一直形同虚设，团队没有一条稳定的代码分支。

这个问题很大程度上源于团队对分支策略的不了解，本文就简单聊一聊Git中的工作流——分支策略。

# 常见的分支策略

上文中提到的团队使用分支策略很混乱，这种分支策略也并不是主流的，使用主流的分支策略则会避免以上问题。

常见的分支策略有以下三种：GitFlow、GitHubFlow以及GitLabFlow。

## Git Flow

GitFlow是这三种分支策略中最早出现的。

GitFlow通常包含五种类型的分支：Master分支、Develop分支、Feature分支、Release分支以及Hotfix分支。

- **Master分支**：主干分支，也是正式发布版本的分支，其包含可以部署到生产环境中的代码，通常情况下只允许其他分支将代码合入，不允许向Master分支直接提交代码（对应生产环境）。

- **Develop分支**：开发分支，用来集成测试最新合入的开发成果，包含要发布到下一个Release的代码（对应开发环境）。
- **Feature分支**：特性分支，通常从Develop分支拉出，每个新特性的开发对应一个特性分支，用于开发人员提交代码并进行自测。自测完成后，会将Feature分支的代码合并至Develop分支，进入下一个Release。
- **Release分支**：发布分支，发布新版本时，基于Develop分支创建，发布完成后，合并到Master和Develop分支（对应集成测试环境）。
- **Hot**  **fix分支**：热修复分支，生产环境发现新Bug时创建的临时分支，问题验证通过后，合并到Master和Develop分支。

通常开发过程中新特性的开发过程如下：

从Develop分支拉取一条Feature分支，开发团队在Feature分支上进行新功能开发；开发完成后，将Feature分支合入到Develop分支，并进行开发环境的验证；开发环境验证完成，从Develop分支拉取一条Release分支，到测试环境进行SIT/UAT测试；测试无问题后，可将Develop分支合入Master分支，待发版时，直接将Master分支代码部署到生产环境。

可参考下图：

![Untitled](Git%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E4%B8%89%E7%A7%8D%E5%88%86%E6%94%AF%E7%AD%96%E7%95%A5%EF%BC%9AGitFlow%E3%80%81GitHubFlow%E4%BB%A5%E5%8F%8AGitLabFlow%20e2db9453df9e4fa5a9a276157b4b0cd6/Untitled.png)

GitFlow的优点是每个分支都有明确的定义，严格按照GitFlow管理项目代码的话，很难出现代码混乱；其缺点是：如果特性分支过多的话很容易造成代码冲突，从而提高了合入的成本；由于每次提交都涉及多个分支，所以GitFlow也太不适合提交频率较高的项目。

## GitHubFlow

GitHubFlow看名字也知道和GitHub有关，它来源于GitHub团队的工作实践。当代码托管在GitHub上时，则需要使用GitHubFlow。相比GitFlow而言，GitHubFlow没有那么多分支。

GitHubFlow通常只有一个Master分支是固定的，而且GitHubFlow中的Master分支通常是受保护的，只有特定权限的人才可以向Master分支合入代码。

在GitHubFlow中，新功能开发或修复Bug需要从Master分支拉取一个新分支，在这个新分支上进行代码提交；功能开发完成，开发者创建Pull Request（简称PR），通知源仓库开发者进行代码修改review，确认无误后，将由源仓库开发人员将代码合入Master分支。

![Untitled](Git%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E4%B8%89%E7%A7%8D%E5%88%86%E6%94%AF%E7%AD%96%E7%95%A5%EF%BC%9AGitFlow%E3%80%81GitHubFlow%E4%BB%A5%E5%8F%8AGitLabFlow%20e2db9453df9e4fa5a9a276157b4b0cd6/Untitled%201.png)

很多人可能会问，提交代码通常是commit或者push，拉取代码才是pull，为什么GitHubFlow中提交代码提出的是“Pull Request”。因为在GitHubFlow中，PR是通知其他人员到你的代码库去拉取代码至本地，然后由他们进行最终的提交，所以用“pull”而非“push”。

GitHubFlow优点是相对于GitFlow来说比较简单，其缺点是因为只有一条Master分支，万一代码合入后，由于某些因素Master分支不能立刻发布，就会导致最终发布的版本和计划不同。

## GitLabFlow

GitLabFlow出现的最晚，GitLabFlow是开源工具GitLab推荐的做法。

GitLabFlow支持GitFlow的分支策略，也支持GitHubFlow的“Pull Request”（在GitLabFlow中被称为“Merge Request”）。

相比于GitHubFlow，GitLabFlow增加了对预生产环境和生产环境的管理，即Master分支对应为开发环境的分支，预生产和生产环境由其他分支（如Pre-Production、Production）进行管理。在这种情况下，Master分支是Pre-Production分支的上游，Pre-Production是Production分支的上游；GitLabFlow规定代码必须从上游向下游发展，即新功能或修复Bug时，特性分支的代码测试无误后，必须先合入Master分支，然后才能由Master分支向Pre-Production环境合入，最后由Pre-Production合入到Production。

![Untitled](Git%E5%B7%A5%E4%BD%9C%E6%B5%81%E4%B8%AD%E5%B8%B8%E8%A7%81%E7%9A%84%E4%B8%89%E7%A7%8D%E5%88%86%E6%94%AF%E7%AD%96%E7%95%A5%EF%BC%9AGitFlow%E3%80%81GitHubFlow%E4%BB%A5%E5%8F%8AGitLabFlow%20e2db9453df9e4fa5a9a276157b4b0cd6/Untitled%202.png)

GitLabFlow中的Merge Request是将一个分支合入到另一个分支的请求，通过Merge Request可以对比合入分支和被合入分支的差异，也可以做代码的Review。