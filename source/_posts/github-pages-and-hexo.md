---
title: Hello GitHub Pages and Hexo
date: 2018-12-02 11:30:39
tags:
---

前几天做产品的朋友说他在用 GitHub Pages 来托管 PRD 文档，为了方便文档的共享与更新，遇到一些问题求教如何解决。当真是惊到我了，别人家的产品经理比我们家的程序员还要努力，我要是再不努力就要被产品经理淘汰了。
我自己并没有用过，对前端也不熟悉，但这么酷的需求怎么能放过，心中下定决心一定要解决他的需求，于是周末赶紧抄起键盘开始学习，准备先搭建个自己的 blog，这样边用边学效率更高。

## What?
学习一个东西，我们应该首先先搞清楚是什么，这样才能将这个东西快速归类到自己的知识框架中，以便进一步理解

- GitHub Pages: 静态网站托管服务
- Hexo: 博客框架，支持 **Markdown** 生成博客静态网站，并一键部署到 GitHub Pages

## Why?
Why GitHub Pages?
- 省钱：不用自己租服务器，免费使用
- 省心：不用自己维护服务器与网页服务，不用操心服务稳定性
- 简单：基于 Git 仓库，push 即可

Why Hexo?
- 命令行操作，简单高效，支持一键部署和持续集成
- 支持 Markdown ，文章编辑体验良好
- 主题丰富，颜值正义

## How?

### 配置 [GitHub Pages](https://pages.github.com/)

- 创建 git 仓库，以 `name.github.io` 命名，其中 `name` 可以自定义
- 将静态网页 push 至 git 仓库 `https://github.com/username/name.github.io`
- 浏览器访问 `https://name.github.io` 就能看到网页了！

### 安装 [Hexo](https://hexo.io/) 
安装 Node.js

`brew install node`

安装 Hexo

 `npm install hexo-cli -g`

初始化 Hexo 

到目标文件夹 `<floder>`（一个文件夹对应一个站点）

``` bash
$ hexo init <floder>
$ cd <floder>
$ npm install
```

### 保存源码

GitHub Pages 会托管仓库 master 分支上静态网页
Hexo 部署，只将生成的静态网站 push 到 github 了，源码并没有 push，这样不好维护，比如换台电脑没有源码就无法编辑文章了
如何解决？利用 git 分支，新建 source 分支用于保存源码

``` bash
git checkout -b source
git push origin source
```



- source 分支：项目源码，用于编辑维护
- master 分支：源码生成的静态网页（即源码 `public` 文件夹下的内容）



### 编写

本地预览

- 启动本地服务 `hexo server`
- 浏览器访问 http://localhost:4000 (默认端口号4000)



主题

预览后是不是感觉挺丑的？没有关系，Hexo 支持丰富的主题 [theme](https://hexo.io/themes/) 供我们选择
1.将主题文件放到文件夹 `themes/` 下，建议用 git submodule 的方式引用主题的源码，这样方便更新，也方便 CI 依赖

``` bash
git submodule add https://github.com/next-theme/hexo-theme-next themes/next
```

2.配置主题

再次预览看看，是不是感觉好多了🙂



### 部署

安装 git 部署插件 `npm install hexo-deployer-git --save`

配置 hexo 部署

```yaml
deploy:
  type: git # 类型填git
  repo: <repository url> # Git 仓库地址
  branch: master # 分支。默认是 master 
```



`hexo clean && hexo g -d && hexo s`



### 持续集成自动部署

TODO