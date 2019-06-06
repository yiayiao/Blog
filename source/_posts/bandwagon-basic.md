---
title: 搬瓦工服务器常见的问题
date: 2019-03-03 21:57:30
tags:
    - Blog Setup
categorys: 
    - Linux 
    - Server Management
---

&emsp;&emsp;自己写这个博客，主要是记录一些事情，首先是把事情记录下来，然后再考虑把叙事讲清楚。这便是一篇重在记录的博客，在这里记录了一些使用搬瓦工服务器过程中遇到的问题，如果你恰好看到了这篇博客又遇到了相似的问题，希望能帮到你。

### 1. 我的搬瓦工被墙了！

&emsp;&emsp;某一天，忽然发现，出于国家对我的保护，我的搬瓦工服务器没法访问了；然而因为我们国土很大，人很多，这种事情国家没法及时通知到我，而我又是第一次遇到这种情况，所以瞬间慌了神。不过我还是很快意识到自己的服务器IP可能被墙了，于是我首先对自己的行为进行了反思，我从未发布过不利于国家安定团结的信息，也没用自己的服务器干过坏事，所以我的IP地址被墙，应该只是一场误会；经过反思，我放下了一百个心。下一次如果你的服务器也被墙了，也先反思反思，即使是国外服务器，也不意味着自己想干啥就能干啥。为了指导你进一步进行反思，这里我们再给出一个链接：{% link 为什么搬瓦工VPS主机IP容易封 https://banwagong.cn/why-feng-ip.html install %}，帮助你进行更专业的反思。

&emsp;&emsp;既然自己没有做过坏事，现在找回自己搬瓦工服务器！首先我们确认自己的服务器确实被墙了，可以参考下面这篇教程，写得很不错：{% link 怎样检查搬瓦工IP被墙 https://www.banwago.com/1265.html install %}，简而言之就是通过两个专业的网站：{% link 站长工具 http://ping.chinaz.com/ install %}和{% link ping.pe http://ping.pe/ install %}对自己的IP地址进行ping操作。

&emsp;&emsp;搬瓦工官网上也能检测自己的IP是不是被墙了，但是这个法子并不是很靠谱，可能会误报。搬瓦工确实是一款良心产品，允许你每10个星期免费更换一次IP，前提是你的IP地址确实被墙了。在官网上检测IP是否被墙和更换免费IP是同一个URL，点击“{% link 更换免费IP https://kiwivm.64clouds.com/main-exec.php?mode=blacklistcheck install %}”直达，一个非常小清新的页面，不说你也知道该点哪：

{% asset_img block_list_check.PNG [官网测试自己的IP是否被墙] %}

&emsp;&emsp;点击Test Main IP进行测试，如果测出你的IP被墙了，网页下方会出现一个更换IP地址的链接，点击可申请一个新的IP地址。这个网页的登录做得不太人性，需要输入“Server IP”和密码，而不是用户名和密码，可能是为了方便名下有多个服务器的土豪进行管理，但是敢问有几个人真的能记住自己的“Server IP”呢，反正我一直记不住。

### 2. 我的搬瓦工太慢了!

&emsp;&emsp;一只都有这样的困扰，自己的搬瓦工服务器很慢，尤其是使用家里的移动宽带时，播放视频往往得用最低的清晰度，使用终端去连接经常需要重试好几次。通过搜索的尝试，总结了以下三个手段，可以提升搬瓦工服务器的访问速度，它们分别是BBR、FinalSpeed和net_speeder。

#### 2.1 BBR

&emsp;&emsp;管理搬瓦工服务器，在选择系统进行安装时，选择带后缀bbr的，安装完成之后，BBR应该就是默认开启的。你可能和我一样，过了一段时间之后，已经忘了自己的服务器是否在已经在安装系统了已经支持BBR了，所以建议在安装之前，先使用如下的命令检查一下。

{% codeblock %}
lsmod | grep bbr
{% endcodeblock %}

&emsp;&emsp;返回值有tcp_bbr模块则说明BBR已经启动。

&emsp;&emsp;如果检测到BBR没有启动，你则需要安装BBR，简而言之，使用BBR包含两部操作，升级Ubuntu内核，然后配置/etc/sysctl.conf系统文件。去谷歌搜索一下“Ubuntu安装BBR”，能搜出很多教程，在这里我介绍一个相对简单的方法，有牛人将BBR安装配置操作整理为了一份一键安装脚本，极大方便了BBR的安装，操作如下：

{% codeblock %}
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
{% endcodeblock %}

&emsp;&emsp;详情参考：{% link 秋水逸冰的专业教程 https://teddysun.com/489.html config_bbr %}

#### 2.2 net_speeder

&emsp;&emsp;net_speeder采用了多倍发包的方式，用流量换速度，提升了搬瓦工服务器的访问效率。先给出一篇写得很不错的net_speeder安装教程：{% link 厘米天空 https://www.cmsky.com/vps-net-speeder/ config net_speeder %} ，我操作的时候基本参考了厘米天空的这篇教程。

&emsp;&emsp;net_speeder在Ubuntu上可以通过下面的命令进行一键编译：

{% codeblock %}
wget --no-check-certificate https://raw.githubusercontent.com/tennfy/debian_netspeeder_tennfy/master/debian_netspeeder_tennfy.sh
chmod a+x debian_netspeeder_tennfy.sh
bash debian_netspeeder_tennfy.sh
{% endcodeblock %}

&emsp;&emsp;注意只是编译，而不是安装，上面的命令执行完成之后，会在当前路径下编译出一个net_speeder文件，建议将编译出来的文件放到/usr/local/bin/目录下，方便执行。

* 启动net_speeder：

{% codeblock %}
net_speeder eth0 "ip"
#这里的eth0需要替换为自己服务器的网卡名称，而"ip"千万别自作主张替换为了本机的IP地址；
#使用以下的命令则可以指定对某个端口进行加速
net_speeder eth0 "tcp src port 8388 or port 8389"
{% endcodeblock %}

* 关闭net_speeder：

{% codeblock %}
#简单明了的系统命令
killall net_speeder
{% endcodeblock %}

&emsp;&emsp;然后可以根据自己的需要，添加开机启动，或者添加crontab定时任务，让net_speeder在固定的时间开启，在固定的时间关闭，请参考{% link 厘米天空 https://www.cmsky.com/vps-net-speeder/ config net_speeder %}的教程。


#### 2.3 FinalSpeed

&emsp;&emsp;写到这里还有一点激动，当FinalSpeed安装完成，可以明显感受到通过搬瓦工访问视频网站的速度获得了极大的提升。不同于BBR和net_speeder，FinalSpeed除了需要在服务器上启动服务之外，还需要在本地安装FinalSpeed客户端，这给FinalSpeed的使用带来了很大的局限，而在手机上使用FinalSpeed当前是不可能的。

&emsp;&emsp;借鉴了别人的教程，首先还是要给出参考的：{% link 教程连接 https://www.wervps.com/we/1089.html config FinalSpeed %} 。在服务器上安装FinalSpeed，也有一键式的命令：

* 一键安装：

{% codeblock %}
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/finalspeed/master/install_fs.sh && bash install_fs.sh
{% endcodeblock %}

* 一键卸载：

{% codeblock %}
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/finalspeed/master/install_fs.sh && bash install_fs.sh uninstall
{% endcodeblock %}

* 操作命令：

{% codeblock %}
#启动：
/etc/init.d/finalspeed start
#停止命令：
/etc/init.d/finalspeed stop
#查看状态：
/etc/init.d/finalspeed status
#查看日志：
tail -f /fs/server.log
{% endcodeblock %}

&emsp;&emsp;FinalSpeed依赖了Java，如果搬瓦工服务器上没有安装Java，FinalSpeed安装时会报错，Java的安装也很简单，先在自己的终端上敲一个java命令，然后按照报错提示安装相应的java版本就行了。上文提到的安装教程还有一个不足之处，我自测时，上文链接到的finalspeed_install1.0的客户端版本已经不可用了，给出finalspeed_install1.2客户端下载链接：{% link finalspeed_install1.2.exe https://raw.githubusercontent.com/91yun/finalspeed/master/finalspeed_install1.12.exe config net_speeder %}

&emsp;&emsp;客户端的配置还是可以参考前面的教程，这里给出我配置完成之后的net_speeder客户端截图：

{% asset_img finalspeed_client.PNG [FinalSpeed客户端配置] %}

&emsp;&emsp;物理宽带的下载速度和上传速度请按照自己实际的网速进行配置。配置完成之后，可以通过本地IP加本地端口的方式，访问服务器IP的加速端口。

&emsp;&emsp;写在最后，当记录某项操作时，我希望自己能弄清楚背后的原理，无奈之BBR、FinalSpeed和net_speeder三者间，之后net_speeder比较容易理解，理解BBR和FinalSpeed都需要掌握相当的计算机网络知识，自己一时半会没法深入理解，所以先在这里立下一个Flag，后面再补上BBR和FinalSpeed的讲解。在此先感谢为我们创建了这些好用的工具的程序员。


