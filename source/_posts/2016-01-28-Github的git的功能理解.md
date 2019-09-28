---
date: 2016-01-28
layout: post
title: Github的git功能的理解
category: Other
---



Github 我之前一直当做SVN来用了，不过也难怪，对这些代码管理工具并不是很熟悉，今天又看了Github的官方[十分钟教程](https://guides.github.com/activities/hello-world/)，加深了一下Github功能的理解。



## Repository

Repository 就是一个代码仓库，我们所有的工作都是围绕着它展开。



## Issue

用来发布重要的问题，例如bug反馈和讨论，或者feature，或者分工。



## Branch

branch 是分支，一般用于团队合作，也可以用于一个产品进行功能的开发分支，也可以理解为多个团队共同维护一个产品，但是不通团队负责产品的不同功能开发。

但是`master`是默认的主分支，只能有一个，其他新创建的branch 可以合并到`master`分支上。



## Commit

commit 是确认这次所有的修改，并且在commit 的时候加上这次commit的 description。



## Pull Request

这是`Fork`项目的用户请求贡献代码，通过Pull Request 请求向仓库的管理员请求合并代码。并且这个阶段会显示出冲突的地方，需要fix。



## Merge

Merge阶段是对Pull Request 进行处理的阶段，审核并且合并到相应的分支中。
