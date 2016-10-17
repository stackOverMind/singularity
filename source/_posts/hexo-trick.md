---
title: hexo-trick
date: 2016-10-17 21:13:17
tags: [hexo]
---
# hexo

hexo 是一套强大而简单的博客系统，能够生成静态网页部署到任何容器中。而且有很多现成的theme可选，纯nodejs实现。

```
# 全局安装hexo
npm install -g hexo-cli 
# 建站
hexo init <forder>
# 新建一篇文章，在 source/_posts/目录下可以编辑这篇文章了
hexo new post <title>
# 生成静态网站
hexo generate
```
详情移步[官方文档](https://hexo.io/zh-cn/docs/)

hexo 在本地玩6了之后我们就想找个地方部署起来，土豪的路子是到阿里云租个vps，然后装个nginx，不过现在是云服务的时代了，能轻则轻，仅仅为了搭建个博客买个阿里云有点太重，跟钱没啥关系。

下面我介绍一个免费。。啊不对。。轻量级的解决方案：

# hexo 结合 travis CI，github pages 做自动发布
如果你要搭建一个个人博客，然而你想免费，那么这篇文章非常适合你，因为我要介绍一种永久免费的博客搭建方式，尤其适合程序员。因为用到的工具就是开发常用的工具
* github pages 对于每一个用户，github慷慨的给予一个域名（xxx.github.io）来托管你的静态网站
* travis CI 一套CI系统，平时用来做自动化测试，其实凡是自动化测试的工具都不止可以做测试，能够做很多脑洞大开的事情，甚至搞破坏。travis CI能够做到在云端跑一段测试代码，那么我们可以用这段测试代码来把代码不熟到github pages 上。

基本思想就是这样，详细过程之后补充。。