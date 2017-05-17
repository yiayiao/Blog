---
title: Vim Installation on Ubuntu
date: 2017-02-12 13:35:21
tags: Installation
---
Vim is one of the most fantastic editors for coding. Once I have installed the Ubuntu system, the next step will be the installation or configuration of Vim. Good configuration takes much time, but finally you will find it's worth to do that. Here I record how I configure my vim, just make a simple record in case that I will configure it again, and may you, my dear reader, like it!

## Wonderful Guide from Github

Here is a wonderful guide I followed to configure my vim: [ma6174](https://github.com/ma6174/vim)
Just follow the guide you will get a get a well configed vim, with many useful plug-ins which can help you show the file structure, autocomplete the code, do code checks and more. But when I tried on my computer, there was one shortpoint about this guide, if you want to know it, keep on reading.

## Replace Pyflakes with Flake8

Once you have followed the guide and finished the installation of vim, you can try this command:

``` bash
vim test.py
```

If everything is OK and there is no error on you terminal page, congratulations, enjoy your Vim. But during my configuration, I got a error:

![error](pyflakes-error.png) 

When I searched the internet, there was a page written that pyflakes is obsoleting, and flake8 was suggested to replace pyflakes. So I removed the plug-in pyflakes and then installed flake8.

* #### How to Remove Pyflakes:

``` bash
rm ~/.vim/ftplugin/python/pyflakes.vim
```

* #### How to Install Flake8:

Just give a link here, follow this guide and you can manage it yourself: [flake8](http://blog.csdn.net/roy9494/article/details/17439069)