---
title: Ubuntu安装使用MariaDB
date: 2019-03-31 11:09:23
tags:
 - Installation
categories: 
 - Database
 - Maintenance
---

在Ubuntu上安装MariaDB的经历，其挫折系数仅次于数年前安装Oracle。然而安装完成之后再回头看，其实也并没有那么复杂，搞出这么多幺蛾子，还是自己因为自己对MySQL和MariaDB一知半解，吃了没文化的亏。

MadiaDB数据库是MySQL数据库的一个非常活跃的分支，为什么要安装MariaDB，因为MySQL被Oracle公司收购之后，更新节奏变慢，而且有被Oracle闭源的风险。所以，是时候跟着MySQL之父Michael Widenius一起拥抱MariaDB了。

我的Ubuntu系统为Ubuntu 18.04 LTS，安装的MariaDB版本为**10.3**版本，下面给出安装命令：

{% codeblock %}
#安装依赖包
apt install software-properties-common -y
#添加密钥
apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
#添加PPA源
add-apt-repository 'deb [arch=amd64] http://mirrors.tuna.tsinghua.edu.cn/mariadb/repo/10.3/ubuntu bionic main'
#安装mariadb-server
apt update
apt install mariadb-server -y
{% endcodeblock %}

*如果你想要安装10.2, 10.1, 5.5等其他版本的MariaDB，请将命令中的10.3替换为对应的版本号*

怎么样，是不是很简单，那我怎么就掉进了坑里了？怎么掉进去的我也不知道，当我重启电脑之后，发现MariaDB数据库没起来。我尝试用“systemctl start mysql”命令启动MariaDB，启动了一段时间之后启动失败了，根据提示使用“journalctl -xe”查看错误日志，发现日志中如下的错误：

{% codeblock %}
kernel: [ 2336.792423] audit: type=1400 audit(1470265086.730:518): apparmor="DENIED" operation="sendmsg" info="Failed name lookup - disconnected path" error=-13 profile="/usr/sbin/mysqld" name="run/systemd/notify" pid=11850 comm="mysqld" requested_mask="w" denied_mask="w" fsuid=117 ouid=0
{% endcodeblock %}

从日志里面看，MariaDB启动失败和apparmor有关，apparmor是什么东西，遇到这个问题之前我不懂，因为这个问题我简单了解了一下，apparmor安全策略用于定义个别程序可以访问的系统资源以及各自的特权，简单理解一下，是一个做安全配置的东西。当我根据网上搜来的一些指导去编辑MySQL的安全配置文件“/etc/apparmor.d/usr.sbin.mysqld”时，我注意到文件内容是空的，仅有如下一段注释说明：

>\# <b>This file is intentionally empty to disable apparmor by default</b> for newer
>\# versions of MariaDB, while providing seamless upgrade from older versions
>\# and from mysql, where apparmor is used.
>\#
>\# By default, <b>we do not want to have any apparmor profile for the MariaDB</b>
>\# server. It does not provide much useful functionality/security, and causes
>\# several problems for users who often are not even aware that apparmor
>\# exists and runs on their system.
>\#
>\# Users can modify and maintain their own profile, and in this case it will
>\# be used.
>\#
>\# When upgrading from previous version, users who modified the profile
>\# will be prompted to keep or discard it, while for default installs
>\# we will automatically disable the profile.

注意上文中被我加粗的内容，MariaDB默认是关闭了apparmor策略的，那为什么还是被apparmor搞得启都启不来了，一脸懵逼。随后我继续尝试搜索解决方案，按照网上的一些教程对apparmor配置文件进行了配置，取得了一些进展，不过仅限于日志中的错误信息发生了变化。

最后还是在{% link stackoverflow https://stackoverflow.com/questions/22473830/docker-and-mysql-libz-so-1-cannot-open-shared-object-file-permission-denied solve %}中找到了解决方法，解决方法还是关闭MariaDB的apparmor配置，使用下面的方法：

{% codeblock %}
#Type this on your host terminal
sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.mysqld
{% endcodeblock %}

在博客园的一篇博客中，找到了对上述操作的解释，/etc/apparmor.d/disable目录可以和apparmor_parser -R选项一起使用以禁用一个配置文件，这里给出{% link 链接 http://www.cnblogs.com/popsuper1982/p/3818116.html solve %}。

写这篇博客时，我尝试将它写得悲壮一点，像读者传递一下解决问题过程中的艰辛，但是写博客的过程中，我渐渐放弃了这个想法，还是写得朴实一点，把事情写清楚最重要。对于我们大多数程序猿来说，解决问题就是我们的日常，每当解决了一个问题，就积累了一些经验，所以面对问题，我们应该抱有积极的心态。在解决这个问题的过程中，我将MariaDB翻来覆去安装了很多遍，配置文件改了又改，在MySQL的配置，以及日志的查看等方面有学到了一些知识。

最后，在解决MariaDB安装的过程中，还有一个意外的收获，解决了Ubuntu上网易云音乐每次启动都需要重新登录的问题，原因就在“.cache/netease-cloud-music/”目录下有一些文件的属主是root，用“chown -R”命令修改整个目录的属主为自己的用户之后，问题就解决了！

