---
title: hexo备份
date: 2022-10-23 21:42:28
toc: true
tags: other
categories:
  - hexo
---

# Hexo博客如何备份到Github

## Hexo博客备份

### 创建新分支

在`Github`上博客仓库下创建一个新的分支`hexo`，并且将这个分支设置为默认分支，具体操作如下：

![img](hexo%E5%A4%87%E4%BB%BD/1642508-61f52f7ed4251885.png)

20220511_03.png



![img](hexo%E5%A4%87%E4%BB%BD/1642508-18cbfa4d2b1dac53.png)

20220511_04.png



![img](hexo%E5%A4%87%E4%BB%BD/1642508-b13c0790e40c52ae.png)

20220511_05.png



### 克隆hexo分支

在本地把我们刚建的分支`hexo`克隆到本地：

![img](hexo%E5%A4%87%E4%BB%BD/1642508-e87f19cfc93ffa94.png)

20220511_06.png


把克隆下来的项目里面的`.git`文件复制到我们的Hexo博客目录下:

![img](hexo%E5%A4%87%E4%BB%BD/1642508-42e7569d97440356.png)

20220511_07.png


**注意**：如果之前搭建博客的时候自己更换过主题文件的，请把主题文件里面的`.git`文件删除。



### 开始备份

进入到Blogs根目录下，执行如下命令：



```bash
git add .
git commit -m "Blog源文件备份"
git push origin hexo
```

这时候我们会看到`Github`上的`hexo` 分支就有我们的源文件了。

![img](hexo%E5%A4%87%E4%BB%BD/1642508-4fb324fec7446605.png)

20220511_08.png


如果你想要每次更改东西都希望备份到`hexo` 分支上，可以执行如下步骤：





```csharp
hexo clean
git add .
git commit -m "备份"
git push
hexo g & hexo d
```

## 如何恢复博客

假如我们现在更换了电脑，希望在新的电脑上继续写博客，把`Github`上`hexo`分支上的项目克隆到本地（注意：是我们备份的那个分支）

进入到克隆下来的文件夹，执行如下命令：



```undefined
npm install hexo-cli
npm install hexo-deployer-git
```

然后再去安装主题相关的插件即可，当然如果你电脑上还没有 `Node.js`等环境的话可能还需要去安装相关的环境。
现在我们就基本上可以在另一台电脑上继续我们的博客之旅啦～