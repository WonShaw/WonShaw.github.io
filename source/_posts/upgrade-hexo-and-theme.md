---
title: 更新hexo及主题
date: 2019-11-21 18:24:04
tags:
---


记录一下升级hexo和主题的过程。

任何软件都存在一定的安全缺陷，github会对使用hexo作为GitHub pages的仓库发出安全警告，同时发出Pull Request（PR）协助修复。

其实直接对GitHub发出的PR进行合并并不是好办法，因为hexo本身是一个软件，自身更新里就包含了对安全缺陷的修复，GitHub的自动PR只是hexo官方修改的一个很小的子集，因此最好的办法是直接更新hexo。

以我的仓库为例，首先进入blog根目录，使用以下指令查看哪些依赖需要更新

```bash
npm outdated
```

执行后会有相关的提示，告知Latest版本。

打开`package.json`文件，将里面对应的需要更新的依赖的版本修改成上面给出的最新版本。

保存后，执行

```bash
npm install
```

更新完成后，可以通过以下指令检查是否成功

```bash
hexo -v
```

接下来执行以下指令，在本地看看效果

```bash
hexo g
hexo s
```

由于更新的版本跨度略大，我所使用的next主题显示略有问题，因此需要更新主题。

由于我的主题是直接以submodule的形式管理的（参考[主题管理](https://wonshaw.github.io/2019/06/use-hexo-on-osx/#主题管理)），因此让submodule更新到最新的提交即可。

```bash
git submodule update --remote
```

再次本地生成文档预览，没有问题，接下来执行以下指令发布即可。

```bash
hexo clean
hexo d
```

由于发布的只是静态站点，最好把源文件也托管了，比如我的GitHub仓库里hexo分支就是源文件，只需要把本地修改（也就是更新package.json和主题的修改）提交并push上去即可（参考[源文件管理](https://wonshaw.github.io/2019/06/use-hexo-on-osx/#源文件管理)）