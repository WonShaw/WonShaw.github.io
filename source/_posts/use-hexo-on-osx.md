---
title: 使用hexo构建博客
date: 2019-06-29 02:46:28
tags:
---



github pages可以托管前端页面，配合hexo，就可以搭建一个博客系统。

当然这个博客系统和平时我们使用的博客还是有很大不同的，流程上，我们需要在本地，写好文章，使用hexo生成静态网页，将静态网页上传github仓库，如果之后有任何修改，都需要先在本地改好，再上传github。当然，源文件我们也可以上传该仓库，使用一个单独的分支即可。

作为生产工具，我还是喜欢MBP，因此我下面的操作都是在OS X上完成的。

### 安装nodejs

首先到[nodejs官网](https://nodejs.org/en/)下载nodejs并安装，这一步需要翻墙，当然也可以用淘宝做的站点[nodejs中文网](http://nodejs.cn/download/)，这个可以直接下载。

### 配置npm

npm会随着nodejs一起安装，但由于npm的服务整体被墙了，还是需要借用淘宝的镜像：

```bash
npm config set registry https://registry.npm.taobao.org
```
这样之后npm安装东西时，不至于因为网络问题骂人了。

### 安装hexo

执行以下命令来安装hexo。注意本文这种方式，安装时sudo不可省略：

```bash
sudo npm install -g hexo-cli
```

### 建站

安装完毕后，找个地方执行以下指令建站，下面的指令会生成一个blog目录：

```bash
hexo init blog
cd blog
npm install
```

### 建立github仓库

在github上面建立好github.io的仓库，配置好ssh公钥，本地git配置好，这些都是基本操作，这里不赘述了。

### 配置部署

在blog目录执行下列指令：

```bash
npm install hexo-deployer-git --save
```

在blog目录里，编辑`_config.yml`，找到其中的depoly部分，填写好部署配置：

```
deploy:
  type: git
  repo: <你的github.io仓库 例如git@github.com:WonShaw/WonShaw.github.io.git>
  branch: master
```

这里branch只能是master，github规定用户的github pages仓库只能从master生成网页。

### 编辑及部署

通过`hexo new <md文件名, 无后缀>`创建文章。

通过`hexo g`生成静态网页。

通过`hexo s`在本地生成一个网页服务来预览，浏览器里面访问地址是可以实时看到文章的修改的。

如果要实际进行部署，也就是推送到github上面，先执行清理再部署：

```bash
hexo clean
hexo d
```

### 源文件管理

由于我们push上去的只是静态网址，并不是源文件，如果想管理源文件，可以在github pages仓库开一个分支，将当前目录里的内容push到新建的分支上面去，同时也建议将默认分支改成它，方便在其他地方编辑。

### 主题管理

大部分hexo的主题都是直接clone了整个仓库下来，一般都是：

```bash
git clone git@github.com:xx/xx.git themes/xx
```

然后在`_config.yml`中改主题即可。

但这样会导致主题无法被git管理起来，可以用`git submodule`来管理：

```bash
git submodule add git@github.com:xx/xx.git themes/xx
```

### git走代理

由于目前ssh受到严重的阻断，因此git几乎无法与github交互，因此需要git能走代理，但git使用的ssh，因此本质上要求ssh走代理，一般配置ssh私钥时，有一个config文件，可以直接在里面制定ssh走Shadowsocks在本地的socks5代理

```
Host github.com
    HostName github.com
    User git
    ProxyCommand nc -X 5 -x 127.0.0.1:1086 %h %p
    IdentityFile ~/.ssh/github_rsa
```

注意Shadowsocks客户端可能会在本地开socks5和http代理，两个端口是不一样的，这里要的是socks5代理。

关于SSR，可以购买他人的服务，也可以参考[这篇文章自建SSR](https://wonshaw.github.io/2019/06/deploy_ssr_on_ubuntu/)