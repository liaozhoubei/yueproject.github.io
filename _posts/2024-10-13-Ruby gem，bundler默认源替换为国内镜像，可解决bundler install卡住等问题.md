---
layout: post
title:  "Ruby gem，bundler默认源替换为国内镜像，可解决bundler install卡住等问题"
date:   2024-10-13 01:51:34 +0800
categories: 杂项
---

国内使用bundler install等命令时，经常出现卡住或响应慢的现象，替换为国内源即可解决问题。

# gem

    # 添加 TUNA 源并移除默认源
    gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
    # 列出已有源
    gem sources -l
    # 应该只有 TUNA 一个

# bundler

使用以下命令替换 bundler 默认源

    bundle config mirror.https://rubygems.org https://mirrors.tuna.tsinghua.edu.cn/rubygems

> 官方文档：https://bundler.io/v2.2/man/bundle-config.1.html#MIRRORS-OF-GEM-SOURCES

本文转自 <https://www.jianshu.com/p/28d396a0f05f>，如有侵权，请联系删除。