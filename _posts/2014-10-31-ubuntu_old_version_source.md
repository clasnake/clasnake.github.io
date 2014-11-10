---
layout: post
title: 旧版本Ubuntu源更新
category: technology
---

更新源问题是Debian系统使用同学总是遇到的一大坑。使用Ubuntu12.10三年，但是今年5月16号已经停止线上更新支持了，如果既不想升级系统又希望继续使用更新源，可以令源地址指向旧版本的dist。

<!--break-->

修改`/etc/apt/sources.list`，将所有[http://archive.ubuntu.com/ubuntu](http://archive.ubuntu.com/ubuntu)替换为[http://old-releases.ubuntu.com/ubuntu](http://old-releases.ubuntu.com/ubuntu)即可。

![]({{site:url}}/assets/images/posts/2014-10-31-ubuntu_old_version_source/sources.PNG)