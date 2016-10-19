---
title: hexo 使用心得
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

<!-- more -->
如果你要搭建一个个人博客，然而你想免费，那么这篇文章非常适合你，因为我要介绍一种永久免费的博客搭建方式，尤其适合程序员。因为用到的工具就是开发常用的工具

这里用到两个神器：

* [github pages](https://pages.github.com/): 对于任何一个github用户或者组织，github都允许通过`<your-name>.github.io`这个域名访问你的以 `<your-name>.github.io` 命名的repo，所以你只需要新建一个 `<your-name>.github.io` 的repo就可以了，比如，我的叫做 `stackovermind.github.io`。
* [travis CI](https://travis-ci.org/): travis 本身是收费的商业服务，但是对于github上的开源项目永久免费，所以可以放心使用

一般情况下如果我们使用hexo + github pages的方式搭建个人博客可以用以下的路径
1. 本地使用hexo生成静态文件(hexo generate)
2. 将静态文件目录拷贝出来并且提交到你的 github.io库，甚至hexo提供了配置方式，可以一键部署`hexo deploy`

但这要求你的pc上有node，有hexo。假如没有这些怎么办？假如在公共的电脑上，只有浏览器你将没有办法发布博客。travis CI这时候派上用场。整个流程变成这个样子：

1. 准备两个repo，一个是源码（任意起名字，我的叫做singularity别问为什么），另一个就是上面说的`<your-name>.github.io`
2. 每次往源码库提交的时候自动触发travis 任务运行。
3. travis 执行 `hexo generate` 和  `hexo deploy`的工作
4. 完成

所以重点就是2-3两步如何配置

## 配置自动触发traivs运行
首先，你需要配置一个webhook,让每次提交都触发traivs CI　的执行。
[travis CI](https://travis-ci.org/) 使用你的github账号登陆，你会看到自己的所有库，如果看不到就sync一下。在你的源码库上开启travis就可以。

## 配置travis执行脚本
现在每次更新travis都会自动运行。wait！运行什么? 你还没有告诉travis在什么环境下做什么事情呢！如何告诉呢？在你的项目最顶层增加一个.travis.yml 的文件。

比如我的是这个样子的

```yml
language: node_js
node_js:
- '6'
- '6.1'
branches:
  only:
  - master

cache:
  directories:
    - node_modules
before_install:
- openssl aes-256-cbc -K $encrypted_e69ed82d2201_key -iv $encrypted_e69ed82d2201_iv -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
- cp ssh_config ~/.ssh/config
- git config --global user.name '<your-github-username>'
- git config --global user.email '<your-email>'
install:
- npm install hexo-cli -g
- npm install

script: hexo generate
after_success:
- hexo deploy
```
其他不需要做过多解释，只是before　install做的事情需要多说一下，权限一节详细说。

## 配置hexo的deploy
travis 的执行脚本中要执行　hexo deploy,那么到底如何 deploy 呢？
编辑`_config.yml`

```yml
#上面的都不重要,看下面的

deploy:
    type: git
    repo: git@github.com:stackOverMind/<your-github-username>.github.io.git
```

## 权限
通过上面的配置，travis已经知道该怎么做了，剩下最后一个问题: 权限。
travis其实充当了一台linux pc的角色，在github看来是一个普通的客户端而已，这个时候你需要一个凭证让github相信你是你，然而又不能让其他人伪造这个凭证。

在个人电脑上我们使用的是非对称加密的方式，用`ssh-keygen -t rsa` 生成一对非对称秘钥，其中私钥自己留着（放到 ~/.ssh/下），公钥交给github（复制粘贴到　项目->settings->deploy keys中）。

```sh
$ ssh-keygen -t rsa -C "<your-email>"
```
得到 id_rsa.pub 和 id_rsa。将　id_rsa.pub 复制到　deploy keys

在这里再啰嗦一下非对称加密是怎么回事

* 经过公钥加密的内容只能通过私钥解密，反过来亦然
* 公钥和私钥不同，即使拿到公钥也无法计算出私钥

所以，只要私钥不泄露你就是安全的（不是很严密，不过暂时可以认为安全）

这种方式在travis下就行不通了，因为travis充当了你的私有pc的角色，而github又是公开的，你不能直接把私钥裸着扔到github上，也不能通过网络接口写到travis的服务器上。所以我们需要再加密一次。

travis 提供了命令行工具，但是在安装之前你需要gem，挺蛋疼的。


```sh
$ gem install travis
$ travis login --auto
$ travis encrypt-file id_rsa --add
```
你的`.travis.yml`中会增加跟
｀- openssl aes-256-cbc -K $encrypted_e69ed82d2201_key -iv $encrypted_e69ed82d2201_iv -in id_rsa.enc -out ~/.ssh/id_rsa -d｀　类似的一行。

这时候在你的项目目录中生成了id_rsa.enc 把id_rsa删掉，不要提交到github上，把id_rsa.enc 提上去。
travis执行任务的时候会把这个　id_rsa.enc进行解密，还原成　id_rsa:
```yml
before_install:
- openssl aes-256-cbc -K $encrypted_e69ed82d2201_key -iv $encrypted_e69ed82d2201_iv -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
```


## 其他

为了配置travis 使用ssh 访问github还需要建一个 ssh_config文件。
```
Host github.com
    User git
    StrictHostKeyChecking no
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes
```

这个文件在travis任务执行中被复制到了~/.ssh/中，所以你看，都是套路，只要了解你在linux中怎么做，做成travis脚本就行了。

可以参考这个配置文件
https://github.com/stackOverMind/singularity





