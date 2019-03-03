---
title: Hexo基本操作
date: 2019-02-17 22:44:27
tags: 
    - Blog Setup
---

&emsp;&emsp;这篇博文的名字叫做Hexo的基本操作，可想而知我的博客是通过Hexo搭建起来的。博客托管在了Github Pages上，域名是在阿里云上购买的，为了方便记忆，你可以哼唱一首儿歌：“唐老鸭有一个农村，咦啊咦啊哦”。

&emsp;&emsp;安装Hexo前，需要先安装Git和Node.js，在官网上有详细的指导，Hexo的安装请参见{% link 官网安装指导 https://hexo.io/zh-cn/docs/ install %}。如果希望更详细一些，每一步的截图都展示出来，推荐另外一篇博客：{% link Lixint Blog https://www.lixint.me/hexo-blog.html other %}，这是我见过的最详细的Hexo使用指导。我比较青睐在Ubuntu环境下进行Hexo的安装与使用，在Ubuntu的命令行模式下，无论是安装还是使用，都相比Windows环境更为高效；如果你手上只有Windows，那推荐你使用WSL；如果你只有Windows7，那我推荐你升级到Windows10。

&emsp;&emsp;网上讲Hexo的博客很多，上一段中提到的{% link Lixint Blog https://www.lixint.me/hexo-blog.html  other %}就写的很好；同时Hexo中文版官网写得也不错，很多操作查找起来很方便。本文的目的并不在指导你用会Hexo，而是对一些比较常用的操作进行了记录，起到一个入门和备忘的作用。文中记录的操作都经过了我的亲身实践，本博文持续更新。

### 一些基本的命令

1. 几个和博文写作相关的操作：

|命令|解释|
|----|----|----|
|hexo new <title\> |创建博文|
|hexo new draft <title\> |创建草稿，草稿不会发布到自己的博客上，执行hexo s命令不加-\-draft也无法查看草稿 |
|hexo publish [draft] <title\> |发布草稿，命令后，草稿变成正式的博文| 
|hexo server -\-draft |启动hexo服务器，-\-draft表示预览草稿|

2. 几个和本地server相关的命令

&emsp;&emsp;Hexo官网导航栏中的“服务器”，指的其实是“本地server”，我觉得这个翻译不太好，乍一看还以为是教你怎么把自己写好的博客部署到服务器上，其实它将的是启动本地server相关的操作，这里还是给出官网链接{% link 官网链接 https://hexo.io/zh-cn/docs/server server %}。基本的操作也就那么几个：

|命令|解释|
|----|----|----|
|hexo server | 启动服务器 |
|hexo server -p 5000 | 启动服务器，同时指定端口号，默认为4000 | 
|hexo server -s | 静态模式下启动，只处理public文件夹下的内存，需要先执行 hexo generate|
|hexo server -i 192.168.1.1 | 指定通过哪个IP可以访问，前提是你本地有那个IP地址 |

3. 几个和博文部署相关的命令

|命令|简写|解释|
|----|----|----|
|hexo generate | hexo g | 生成静态文件 |
|hexo generate -\-deploy | hexo g -d | 文件生成后立即部署网站 | 
|hexo deploy | hexo d | 部署网站 |
|hexo deploy -\-generate | hexo d -g | 部署网站，官网上说和hexo generate -\-deploy相同 | 

&emsp;&emsp;你看到上面的几行命令是否也有这样的困惑，“hexo d”命令与“hexo g -d”、“hexo d -g”命令有什么区别，我自己实验后感觉是，没有区别！执行“hexo d”命令，从日志看Hexo就会先生成静态文件，与另外两条命令是一致的。从自己基础阶段常用的Hexo操作来看，“hexo g”这个命令都显得很鸡肋。
&emsp;&emsp;掌握“hexo s”命令，本地博文的任何修改都能实时更新并查看效果，“hexo g -w”和“hexo s -s”命令一起也有与“hexo s”命令一样的效果，却更加繁琐。而掌握“hexo d”一条命令，就能完成自己博文的上传，无需事先执行“hexo g”命令。为了应对可能出现的未知错误，可以再记一条“hexo clean”，在“hexo d”命令前执行。

### 一些常用的配置

不管了，就这样吧，这样打字的时候好用一点，自己的输入法可以快一点点，windows自己的输入法用起来还是怪怪的

试试搜狗输入法，在这里还是不行啊
出来的位置明显的不对
