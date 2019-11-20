---
title: 'solve problem: unable to correct problems, you have held broken packages'
date: 2019-11-04 19:09:33
tags:
    - Linux
    - apt
categories: Linux
# img: http://pzpoejx7j.bkt.clouddn.com/linuxcover.jpg
---

问题原因大多是因为依赖出问题。

最蛋疼的办法是
```bash
sudo apt-get install aptitude
sudo aptitude install <packagename>
```

害，可是都报这个错了，还怎么安装aptitude？？？？？？？？？？？？？？

一般是
```bash
sudo apt-get clean
sudo apt-get update
sudo apt-get upgrade
```

但需要保证镜像源没问题，**本人阿里云和科大云镜像**怎么操作都依旧报错，最后换回初始镜像。
