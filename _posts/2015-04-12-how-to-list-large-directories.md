---
layout: post
title:  "Listing large directories in Linux"
date:   2015-04-12 10:15:30
categories: linux maintainance
comments: true
---

Have you ever wanted to download something and realised there is not enough space left in the computer.

That moment when you realise that there are tens of development repositories in my machine, which means there are lots of `log` directories full of unnecessary files. How do we find them?

This command comes to rescue!

`du --max-depth=7 /* | sort -n`