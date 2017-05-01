---
layout:     post
title:      "【Linux】shell scripts编写上要注意的点"
date:       2017-04-30 19:43:00
author:     "Yuki"
---

这篇博客，持续更新，主要是记录我作为新人踩过的坑...

* if 后面一定要有空格，若是if后面加[]判断的话，[]括号里开头和结束都要有括号

* $引用变量的值，要用{}，例如，引用变量site的值，${site}