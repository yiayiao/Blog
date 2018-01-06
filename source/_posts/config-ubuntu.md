---
title: 我的Ubuntu操作系统配置
date: 2017-12-17 10:03:19
tags:
---
记录自己的Ubuntu踩坑历程。

从上大学算起，接触Ubuntu好多年了，一开始只是跟风，试图把Ubuntu配置得像Windows一样的美观强大，以此证明自己的技术能力。即便装好了，也就是打开Eclipse写写Java程序，或者用vim装模作样写写C，到了大学毕业的时候，还连grep这样基础的Linux命令都敲不顺，真的是年轻，时间多瞎折腾。参加工作之后经常用到Suse服务器，渐渐熟悉了命令行的使用，为了平时在家练手，又把Ubuntu捡了起来，期间还搞了一段时间的黑苹果，经历太黑就不多说了，再用了一年多的Ubuntu，现在已经逐渐习惯打开电脑之后进Ubuntu系统了。

为什么用Ubuntu，这个问题在网上有点像哲学讨论了，对于我而言，首先是工作的影响，Linux的服务器比Windows用的更多，自己渐渐的也比较熟悉了命令行的操作，这个自不用说。然后Ubuntu在24存的显示器上看起来还真的不错啊，挺美观的，看来Ubuntu程序员也喜欢大屏，而且Ubuntu开机很快啊，我通常点睡眠而不是关机，这样启动不到5秒，这样下班回家听两首歌还是不错的；说到听歌，网易云音乐Ubuntu版真是良心之做，补齐了Ubuntu的空挡。最后是Ubuntu给我的感觉，每一个程序自己都熟悉，没有广告忽然冒出来，可以很容易的专注自己的事情，像一个从不多嘴的管家，在这个唠叨的世界，真的很享受这种感觉。不过这个不多嘴的老管家，有时候有点笨，需要你耐心的告诉他要做什么事情。

再说一下这个老管家，他的用户界面就是一坨屎，相比与苹果和Windows，他很丑，而且容易卡，不能要求他像苹果和Windows一样完美，允许和接纳不完美，这也是一种成熟。认识到了Ubuntu用户界面是一坨屎，就不要要求太多，不要乱改主题了，浪费时间，反正不好用。安装程序也是，满足了自己的要求，就一个都不要多装，永远都不求大而全，装一个多一个坑。我会告诉你我写这篇博客时嫌弃默认的中文输入法，为了安装了搜狗输入法耗费了三个小时吗，差点输入法都没了，想为什么费了我三个小时，就往下看吧。

再总结一下上一段，Ubuntu很丑很难用，"just use it, do not optimize it !"

## 你需要知道的Ubuntu小技巧

如果你刚刚接触Ubuntu，这里记录了一些Ubuntu的小技巧，可以帮助提升效率。

1. Ubuntu的Launcher还是不错的，建议把自己常用的应用靠前放，记住它的顺序，这样你可以用“Super + 序号”来启动它，比如我就习惯用“Super+1”打开终端而不是“Ctrl+Alt+T”，因为我的Terminal在Launcher第一个啊。如果觉得Launcher在笔记本上小屏幕上太占空间，可以设置默认隐藏，将灵敏度拉到最低，这样它就不会自己弹出来了，如果想让它出来怎么办，按住Super键就可以啦。

2. 如果真的想要成为Ubuntu用户，应该决心使用Terminal（Linux命令行）完成大部分系统工作，比如最简单的文件拷贝和软件安装，如果不确定这一点，建议就不要在Ubuntu上瞎折腾了，浪费时间。Ubuntu自带的Terminal程序有一些很有用的快捷键配置，比如新建标签页，标签页切换等，你可以配置成自己熟悉的快捷键。

3. 关于Ubuntu的源，网上有很多帖子教怎么修改中国的源，比如阿里的源和163的源，一般是教你修改/etc/apt/sources.list文件，但是我不建议你这么做，最新的Ubuntu系统设置支持选择最快的源。为了安装必要的软件，经常会第三方的库，这时候我也建议用Ubuntu系统设置里面的Software & Updates进行管理。添加了第三方的库之后执行apt-get update可能很慢，可以在系统设置里面把它去勾选，你不需要每次更新它。你还可能遇到安装软件包失败，显示依赖了一个低版本的包，但是系统已经安装了高版本，这也是软件源的问题，自行谷歌修改。

4. 关于操作系统语言的问题，作为一个程序员，最好习惯使用英语界面，如果对汉化有执念，Ubuntu还是让你改的，这样就带来了一个问题，汉化之后的Desktop（桌面）或者Download（下载）这些文件夹也显示为中文，给输入命令带来了一些麻烦。这时怎么办呢，其实也很简单，先把系统切回英语，再切回中文，系统自己会询问是否修改这几个文件夹的名称，你第一次选是，第二次选否，它就是英文的了。


## Ubuntu安装Nemo文件管理器

Ubuntu有自己的文件管理器Nautilus，Nemo和Nautilus相差不大，安装Nemo显然违背了我们能不折腾就不折腾的原则，你大可以跳过本节。我为什么安装Nemo，两个主要原因，首先是Nautilus的图标，一个半开着的抽屉，完全颠覆了我对于文件夹的认知，简单一点画个文件夹不好吗，Nemo的图标正是一个文件夹；其次是nautilus这个单词也太复杂了吧，在命令行里面敲起来难度太大，nemo则简单了很多。

这里我又提到了命令行，你可以尝试一下在命令行里面输入nemo，这时一个nemo窗口便弹了出来，你会发现不管你的当前路劲是什么，输入nemo命令后，打开的都是你的主文件夹。怎么样打开指定的文件夹呢？你想的没错，输入“nemo fileName”，就能在nemo中打开目标文件夹。讲到这里，会很自然地想到怎么打开当前路径，命令也很简单：nemo $(pwd)，可以配置一个别名：

```bash
alias opendir="nemo $(pwd)"
```

将这一行命令添加到~/.bashrc文件中，你就可以通过opendir命令在nemo窗口中打开当前文件夹了。Nemo安装起来也非常的简单，几乎不用折腾，这也是我使用Nemo的原因之一。执行以下命令进行安装：

``` bash
sudo apt-get install nemo nemo-fileroller
```

安装完成后，你可以设置Nemo作为默认的文件管理器：

``` bash
gsettings set org.gnome.desktop.background show-desktop-icons false
```

另外，如果你想使用Nemo图标，输入以下命令：

``` bash
gsettings set org.nemo.desktop show-desktop-icons false
gsettings set org.gnome.desktop.background show-desktop-icons true
xdg-mime default nemo.desktop inode/directory application/x-gnome-saved-search

```

恢复原来的状态，使用下面的命令：

``` bash
gsettings set org.gnome.desktop.background show-desktop-icons true
xdg-mime default nautilus.desktop inode/directory application/x-gnome-saved-search
sudo apt-get remove nemo*
```

给出[原文链接](https://imcn.me/html/y2014/19738.html)


## 配置Vim编辑器




写在最后：不要为了学习C或者C++使用Ubuntu，不要为了学习Python使用Ubuntu，如果你需要在Ubuntu上学习什么，那只可能是Linux系统，
