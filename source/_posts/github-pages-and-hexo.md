---
title: Hello GitHub Pages and Hexo
date: 2018-12-02 11:30:39
tags:
---

前几天做产品的朋友说他在用 GitHub Pages 来托管 PRD 文档，为了方便文档的共享与更新，遇到一些问题求教如何解决。当真是惊到我了，别人家的产品经理比我们家的程序员还要努力，我要是再不努力就要被产品经理淘汰了。
我自己并没有用过，对前端也不熟悉，但这么酷的需求怎么能放过，心中下定决心一定要解决他的需求，于是周末赶紧抄起键盘开始学习，准备先搭建个自己的 blog，这样边用边学效率更高。

## What?
学习一个东西，我们应该首先先搞清楚是什么，这样才能将这个东西快速归类到自己的知识框架中，以便进一步理解

- GitHub Pages: 静态页面托管服务
- Hexo: 博客框架，快速简单的生成博客静态页面

## Why?
Why GitHub Pages?
- 省钱：不用自己租服务器，免费使用
- 省心：不用自己维护服务器与网页服务，不用操心服务稳定性
- 简单：基于 Git 仓库，push 即可

Why Hexo?
- 命令行操作，简单高效
- Markdown 支持，博客编写体验良好
- 主题丰富，颜值正义

## How?
### [GitHub Pages](https://pages.github.com/) 
- 创建 git 仓库，以 `username.github.io` 命名，其中 `username` 可以自定义
- 静态页面推送至 git 仓库 `https://github.com/username/username.github.io`
- 浏览器访问 `https://username.github.io`

### [Hexo](https://hexo.io/) 
- 安装 Node.js [示例：macOS](https://nodejs.org/en/download/package-manager/#macos)
- 安装 Hexo `npm install hexo-cli -g`
- 初始化 Hexo 到目标文件夹 `<floder>`（一个文件夹对应一个站点）
``` bash
$ hexo init <floder>
$ cd <floder>
$ npm install
```
- 本地预览
  - 启动本地服务 `hexo server`
  - 浏览器访问 http://localhost:4000 (服务默认端口号4000)


## Next?
持续集成，自动部署
