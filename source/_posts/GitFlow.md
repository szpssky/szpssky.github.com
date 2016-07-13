---
title: GitFlow工作流
date: 2016-07-07 22:21:54
tags:
- 软件工程
categories:
- Git
comments: true
---
最近没事喜欢逛逛Github，看看热门的开源项目，感觉还是蛮不错的，可以了解到最新的前沿技术，拓展知识面。GitHub之于开源世界如同FaceBook之与互联网社交一般。今天就来分享一下针对Git分布式版本控制系统，在软件开发中所使用的GitFlow工作流。

工作流有各种各样，主要可分为集中式工作流、功能分支工作流、GitFlow工作流、Forking工作流。其中GitFlow工作流可以说是功能分支工作流的升华。

## 何为GitFlow
Gitflow工作流通过为功能开发、发布准备和维护分配独立的分支，让发布迭代过程更流畅。严格的分支模型也为大型项目提供了一些非常必要的结构。
<!-- more -->
Gitflow工作流定义了一个围绕项目发布的严格分支模型，该工作流没有超出分支工作流的概念，但明确的给分支定义了一个角色，并且定义分支与分支之间的交互，如下这张完整的Gitflow工作流示意图。

{% asset_img bigpicture-git-branch-all.png %}
这张图中有5个分支，分别是master、develop、feature、release、hotfixes.
### 历史分支
相对使用仅有的一个master分支，Gitflow分支使用2个分支记录项目的历史，并且建议在任何一个项目开发中无论使用怎样的工作流都应该使用至少1个辅助分支，而不是仅有一个master分支。
develop分支作为功能的集成分支，master作为项目的主分支，项目的整体进度应该是沿着master分支一直向前开发，通常在master分支上使用tag来标记开发的版本号。其余分支都是围绕着这两个分支来进行
### 功能分支
每一个需要开发的新功能，在开发时都应该建立相应的功能分支并且各个功能分支不是从master分支上分出，而是应该在develop分支的基础上分出功能分支，当功能完成是应合并回develop分支。
{% asset_img git-workflow-release-cycle-2feature.png %}
### 发布分支
所谓发布分支是指，当项目完成到一定进度可以发布一个版本时所使用的分支，发布分支也是从develop分支上分出，新建立的这个release分支，作为发布准备所使用，在此时间点之后项目所做的修改也不能再加的这个分支上去了，此分支只可作为为当前发布进行bug修复、文档储存等说使用，如果发布工作已经全部完成则把发布分支合并到master分支并给master打上tag作为一个版本进行发布。当然在release分支建立之后所做的修改此时也应该合并到develop分支中。
{% asset_img git-workflow-release-cycle-3release %}
发布分支的作用就是在准备版本发布时，也不影响开发的继续进行。
### 维护分支
维护分支，在这张工作流图示中即hotfixes分支，该分支仅作为给发布版本进行bug修复使用，这个分支从master分支上分出，修复完成便立即合并回master和develop，并且给master打上新tag分配一个新版本号。

此即为GitFlow工作流的主要内容，在实际开发中，可以结合Pull Request机制进行代码评审(code review)，提高代码质量。

针对Code Review分享一个Google的编程规范[Google Style Guides](https://github.com/google/styleguide)


转载请注明出处，谢谢。