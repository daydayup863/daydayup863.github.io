---
title: 利用Github分支备份Hexo博客源文件  
date: 2020-08-05
tags: [ hexo, github ]
categories: hexo
top: 10
password: 

---

# 场景
Hexo 部署博客很方便，我的这个博客也是用 Hexo 部署在 GitHub Pages 上的，有得人可能在多台电脑上写博客，这个时候需要把博客
的源文件备份在一个地方，这样只需把博客源文件复制下来就可以在另一个地方写博客并部署到 GitHub Pages上了.

本篇介绍的就是利用博客的 repo 分支（ master 分支的必须用来存放你博客网站文件）托管 Hexo 源文件和配置达到备份的目的，
下面开始正题.

<!-- more -->

# 把博客目录的源文件push到repo分支上
cd 进入博客目录，Git 初始化：
```
git init
```

完成之后，添加修改的文件，Hexo 就自带了 .gitignore 文件需要忽略的文件 都已经默认配置好了，add 全部文件：

```
git add .
```
然后commit
```
git commit -m "commit first time"
```

提交成功之后，接下来就是 push 到github了，需要先把这 Hexo 源文件映射到远程 repo 上：
```
git remote add origin https://github.com/your-name/your-name.github.io.git
```

接下来就是把Hexo源文件 push 上去，但是关键的地方到了，master上是 Hexo 生成博客网页的代码，而我们 Hexo 源文件是要 push 到一个分支上面的，所以接下来先要在 repo 上新建一个分支
新建一个叫做hexo分支：
```
git branch hexo
```

查看本地分支，并且切换到 hexo 分支
```
git branch
git checkout hexo
```

再把刚才添加的 Hexo 源文件代码 push 到hexo这个分支
```
git push -u origin hexo
```

然后就可以在 repo 上看到分支里面已经有博客的源文件了


# 日常更新博客源文件

以后你本地的博客源文件的修改就可以直接用 git 命令 push 到 repo 的 hexo 分支上了:
```
git add .  //添加修改内容到本地仓储
git commit -m 'modify blog'  //提交修改内容到本地仓库
git push --set-upstream origin hexo //配置push，以方便后期直接git push推送
git push  //将本地分支和分支下的内容推送到远程

```

注意：执行 git push --set-upstream origin hexo 命令之后，以后修改博客源文件代码之后，直接使用 git push 不用再指定分支，就可以把代码 push 到 hexo 分支上了

# 更换地点使用 repo 分支上的博客源文件

换一台电脑，配置好 Hexo 的环境，配置 Git SSH key，把博客源文件代码克隆下来:
```
git clone xxxxxxxxx.xx (你的 github page 的 repo 地址)
```

博客源文件下载下来之后，默认的分支是 master，需要切换到 hexo 分支
```
git checkout hexo
```

然后cd到博客目录依次执行以下命令：
```
npm install hexo
npm install
npm install hexo-deployer-git --save
```

接下来就可以开始愉快的写博客了，写完之后记得把源文件代码 push 到 Github 上，然后用 Hexo 部署到自己博客上面

