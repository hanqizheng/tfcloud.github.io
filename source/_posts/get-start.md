---
title: get-start
date: 2018-09-28 15:47:47
---
## 快速开始
- fork[同方有云技术博客git仓库](https://github.com/tfcloud/tfcloud.github.io.git)
- 从自己账号上克隆仓库到本地并切换到source分支
- 安装hexo
  > npm i -g hexo-cli
- 用标准脚手架初始化文件
  > $ hexo new standard < title >
- 编辑生成的文件
- 本地提交并push到远程source分支
- 发起pl
  
## 进阶
-  在文件头添加分类和标签（分类按层次排列）
```
---
categories:
- Sports
- Baseball
tags:
- Injury
- Fight
- Shocking
---

```
- 本地查看文章展示效果
```
npm i -g hexo-server

hexo server

```

参阅 [hexo官方文档](https://hexo.io/docs/)