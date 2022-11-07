---
title: hexoguide
date: 2022-10-23 17:19:11
toc: true
tags: other
categories:
  - hexo
excerpt: Hexo 博客常见问题汇总
---



# Hexo 博客常见问题

#### 图片问题

- 参考 https://github.com/cocowool/hexo-image-link
- 安装hexo-image-link

#### 新建新的页面（tags, categories, about）

```hexo new page tags```

新建的页面出现在**souce**文件夹下，修改对应的index.md文件：

```markdown
title: tags
date: 2022-10-24 09:02:56
index: true
layout: tags
type: tags
```

#### ICARUS主题

```shell
$ npm install hexo-theme-icarus
$ hexo config theme icarus
# 配置文件为 _config.icarus.yml
# reference： https://ppoffice.github.io/hexo-theme-icarus/
```

#### 新建博客

``` hexo new name```

# 在Hexo中渲染MathJax数学公式

在用markdown写技术文档时，免不了会碰到数学公式。常用的Markdown编辑器都会集成[Mathjax](https://link.jianshu.com?t=https%3A%2F%2Fwww.mathjax.org%2F)，用来渲染文档中的类Latex格式书写的数学公式。基于Hexo搭建的个人博客，默认情况下渲染数学公式却会出现各种各样的问题。

## 原因

Hexo默认使用"hexo-renderer-marked"引擎渲染网页，该引擎会把一些特殊的markdown符号转换为相应的html标签，比如在markdown语法中，下划线'_'代表斜体，会被渲染引擎处理为`<em>`标签。

因为类Latex格式书写的数学公式下划线 '_' 表示下标，有特殊的含义，如果被强制转换为`<em>`标签，那么MathJax引擎在渲染数学公式的时候就会出错。例如，$x_i$在开始被渲染的时候，处理为$x`<em>`i`</em>`$，这样MathJax引擎就认为该公式有语法错误，因为不会渲染。

类似的语义冲突的符号还包括'*', '{', '}', '\'等。

## 解决方法

解决方案有很多，可以网上搜下，为了节省大家的时间，这里只提供亲身测试过的最靠谱的方法。

更换Hexo的markdown渲染引擎，[hexo-renderer-kramed](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fsun11%2Fhexo-renderer-kramed)引擎是在默认的渲染引擎[hexo-renderer-marked](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fhexojs%2Fhexo-renderer-marked)的基础上修改了一些bug，两者比较接近，也比较轻量级。



```undefined
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

执行上面的命令即可，先卸载原来的渲染引擎，再安装新的。

然后，跟换引擎后行间公式可以正确渲染了，但是这样还没有完全解决问题，行内公式的渲染还是有问题，因为[hexo-renderer-kramed](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fsun11%2Fhexo-renderer-kramed)引擎也有语义冲突的问题。接下来到博客根目录下，找到node_modules\kramed\lib\rules\inline.js，把第11行的escape变量的值做相应的修改：



```ruby
//  escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
  escape: /^\\([`*\[\]()#$+\-.!_>])/
```

这一步是在原基础上取消了对\,{,}的转义(escape)。
 同时把第20行的em变量也要做相应的修改。



```ruby
//  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/
```

重新启动hexo（先clean再generate）,问题完美解决。哦，如果不幸还没解决的话，看看是不是还需要在使用的主题中配置mathjax开关。

## 在主题中开启mathjax开关

如何使用了主题了，别忘了在主题（Theme）中开启mathjax开关，下面以next主题为例，介绍下如何打开mathjax开关。

进入到主题目录，找到_config.yml配置问题，把mathjax默认的false修改为true，具体如下：



```bash
# MathJax Support
mathjax:
  enable: true
  per_page: true
```

别着急，这样还不够，还需要在文章的Front-matter里打开mathjax开关，如下：



```css
---
title: index.html
date: 2016-12-28 21:01:30
tags:
mathjax: true
--
```

不要嫌麻烦，之所以要在文章头里设置开关，是因为考虑只有在用到公式的页面才加载 Mathjax，这样不需要渲染数学公式的页面的访问速度就不会受到影响了。



# Hexo博客设置文章加密

今天给大家推荐一款适用于 Hexo 的静态博客加密插件：hexo-blog-encrypt，搭配此插件你可以写一些比较私密的博客，通过密码验证的方式让其他人不能随意浏览。

1、安装插件
首先运行以下命令，安装设置密码所需要的插件：

```npm install hexo-blog-encrypt```
2、修改配置
在根目录的配置文件_config.yml中添加以下代码：

```shell
encrypt:
  enable: true
```


3、博客文章添加加密字段
设置加密之后，不要忘记在新建博文的时候，在头部添加加密的设置信息，如下图：

```shell
password: 密码
message: 输入密码界面提示说明
```



# Hexo gitee 搭建博客

#### 必须的软件：

**node.js**

node.js官方网站：[http://nodejs.cn/download/](https://ihcll.cn/?golink=aHR0cDovL25vZGVqcy5jbi9kb3dubG9hZC8=)

node.js阿里云镜像站：[https://npm.taobao.org/mirrors/node/](https://ihcll.cn/?golink=aHR0cHM6Ly9ucG0udGFvYmFvLm9yZy9taXJyb3JzL25vZGUv)

**Git**

Git官网(下载速度可能稍慢)：[https://git-scm.com/](https://ihcll.cn/?golink=aHR0cHM6Ly9naXQtc2NtLmNvbS8=)

Git阿里云镜像站（推荐）：[https://npm.taobao.org/mirrors/git-for-windows/](https://ihcll.cn/?golink=aHR0cHM6Ly9ucG0udGFvYmFvLm9yZy9taXJyb3JzL2dpdC1mb3Itd2luZG93cy8=)

#### 可根据个人喜好替换的软件：

**Visual Studio Code**

官方网站：[https://code.visualstudio.com/Download](https://ihcll.cn/?golink=aHR0cHM6Ly9jb2RlLnZpc3VhbHN0dWRpby5jb20vRG93bmxvYWQ=)

**typora**

官方网站：[https://www.typora.io/](https://ihcll.cn/?golink=aHR0cHM6Ly93d3cudHlwb3JhLmlvLw==)

以上软件若你下载速度慢，可以在这篇文章底部找到我已经下载好了的国内链接，链接里不定期更新以上软件的版本（保证均从官网下载下来不做任何修改）。当然最好是自己去官网下，因为我懒，更新不会很频繁，除非你催我一下（发邮件或者其他什么的）……

#### 要用到的框架或平台：

**Hexo**

官方网站：[https://hexo.io/zh-cn/](https://ihcll.cn/?golink=aHR0cHM6Ly9oZXhvLmlvL3poLWNuLw==)

**Gitee（码云）**

官方网站：[https://gitee.com/](https://ihcll.cn/?golink=aHR0cHM6Ly9naXRlZS5jb20v)

#### 安装说明

node.js和Typora就不说了，这个灰常简单。

至于Git，它安装的时候选项挺多的，而且还都是洋文，看不懂的话就全部选 next 就行了。你要是非得搞明白它每个选项的意思，emmmm那建议你们自己去搜吧，网上一搜一大把。我反正是不知道每项的意思，（全是废话…）我也不想知道~/手动狗头

#### 检验是否成功安装

用 Win + R 打开运行，输入 cmd 并进入cmd窗口

##### node.js 的检查

```
 node -v
```

##### npm的检查

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题

```
 npm -v
```

##### Git 的检查

```
 git --version
```

以上有显示版本号，则说明安装成功

#### Hexo的安装

##### 1、安装之前可以先设置一下淘宝镜像加速器

```
 npm install -g cnpm --registry=https://registry.npm.taobao.org
```

##### 2、全局安装框架

```
 npm install hexo-cli -g
```

##### 3-1、创建你的博客目录

```
 hexo init 你博客的文件夹名字
```

##### 3-2、进入你博客的目录

```
 cd 你博客的文件夹名字
```

##### 4、复制文件到你博客的目录

```
 npm install
```

##### 5、安装Hexo部署插件

请在你博客的目录下启动cmd，再执行以下代码

```
npm install hexo-deployer-git --save
```

#### Git的配置

用 Win + R 打开运行，输入 cmd 并进入cmd

##### 设置用户名称

```
git config --global user.name "用户名"
```

##### 设置用户邮箱

```
git config --global user.email "用户邮箱"
```

##### 生成密钥

```
ssh-keygen -t rsa -C "用户邮箱"
```

以上代码执行之后，会让你设置密码，推荐一个都不要设置，直接连按三次回车键。

#### 博客 _config.yml 文件的配置

打开你博客根目录的 _config.yml 文件，将一下信息添加到里面去。

```
deploy:

  type: git

  repo: https://gitee.com/hcllmsx/hcllmsx.git    #替换成你自己仓库的HTTP URL地址

  branch: master
```

【注意区分】你博客根目录的 _config.yml 文件，和主题根目录的 _config.yml 文件！

#### Hexo常用代码

##### 1、清理缓存

```
hexo cl
```

hexo cl 是 hexo clean 的简写

##### 2、生成静态页面

```
hexo g
```

hexo g 是 hexo generate 的简写

##### 3、在本地映射（预览）

```
hexo s
```

hexo s 是 hexo server 的简写

##### 4、部署推送

```
hexo d
```

hexo d 是 hexo deploy 的简写

##### 5、以上连写示例一（清理缓存 + 生成静态页面 + 在本地预览）

```
hexo cl && hexo g && hexo s
```

##### 6、以上连写示例二（清理缓存 + 生成静态页面 + 部署推送）

```
hexo cl && hexo g && hexo d
```

#### 资源下载

由于国内打开国外网站的速度有时候是看运气的，所以如果你下载很慢……可以选择下载我这里下载好的。另外，新建文章的通用模板也在里面，有了它可能会非（mei）常（you）方（luan）便（yong）。最后再再再再重申一次，尽量还是自己下载最新版，因为我更新挺懒的，除非你催我。

链接：[https://msx.lanzoui.com/b0ewfyypa](https://ihcll.cn/?golink=aHR0cHM6Ly9tc3gubGFuem91aS5jb20vYjBld2Z5eXBh)  密码：3t1u