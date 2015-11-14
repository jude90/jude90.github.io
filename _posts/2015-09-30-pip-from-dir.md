---
title: "如何制作一个 pip 离线安装环境" 
layout: post
date: 2015-09-30 17:57:18
---


pip 是python三大神器之一，居家旅行必备。一句 pip -r requirements.txt 就是python 程序员的浪漫。

由俭入奢易，如果遇到项目需要离线搭建环境，就不是一个 pip -r requirements.txt  可以解决问题的。你需要搭建一个本地源。

不必恐慌，制作pip 的本地源也不是一件难事，伟大的社区已经提供了工具，step by step ，本地源会有的。

- 首先你需要一台可以连接其他pip 源的电脑，通常也就是你自己的开发环境，并且安装了pip.

- pip install pip2pi
- 用 pip freeze 在你的开发环境上 制作一个 requirements 文件

- 准备一个 pacakges 文件夹，作为存放本地源的路径

- pip2pi  your_path/packages \
          --no-use-wheel
     -r requirements.txt
- 上面的命令执行完毕后，查看你的packages 目录，会看到所有的包都已经下载下来了。

- 复制packages 目录 和requirements 文件到不能连接外网的目标机器上

- 执行  pip install --no-index --find-links=file:///path_to/packages/ -r requirements.txt
- 离线安装完毕。

参考：

[pip 手册](http://pip-python3.readthedocs.org/en/latest/reference/pip_install.html#options)

[pip2pi 项目](https://github.com/wolever/pip2pi)
