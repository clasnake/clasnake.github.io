---
layout: post
title: su 与 su - 异同
category: 技术
tags: [Linux]
---

shell登录后默认为用户环境，在进行一些特权操作时需要求换到root用户。`su`与`su -`均可用来切换用户。使用`su username`或是`su - username`可切换至相应用户。默认情况下切换至root。然而`su`仅仅切换了用户，而不会切换shell环境；`su -`则不仅切换了用户，同时切换了shell环境。因此如果是刚刚通过`/etc/profile`修改了PATH，仅仅使用`su`是无法看到修改效果的。

<!--break-->

![drawing]({{site:url}}/assets/images/posts/2014-11-10-su_difference/su.PNG)