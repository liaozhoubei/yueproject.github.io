---
layout: post
title:  "Linux环境GitHub推送仓库如何取消每次手动输入登录密码"
date:   2024-10-13 04:51:34 +0800
categories: 杂项
tag: github
---

GitHub仓库连接不再支持网页的登录密码验证
从 2021年8月13日 开始，GitHub 的仓库必须使用 Token 或者 SSH 秘钥验证，不再支持网页的登录密码，否则报错：

	# git push origin main
	Username for 'https://github.com': luoji@gmail.com
	Password for 'https://luoji@gmail.com@github.com': 
	remote: Support for password authentication was removed on August 13, 2021.
	remote: Please see https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
	fatal: Authentication failed for 'https://github.com/luoji/luoji.github.io.git/'

注意！此处 Password 应输入 GitHub设置 中创建的 Token 而非网页登录密码。
创建生成自己的Token
登录GitHub网页端
【Settings】-【 Developer settings】-【Personal access tokens】页面，点击【Generate new token】按钮创建生成新令牌
注意！记得把你新建的 Token 保存下来，Token 只显示在当前页以后无法再查看，若是遗忘只能重新创建。
拉取和推送GitHub仓库

	将 Git 仓库从 GitHub 复制到 本地仓库：
	# git clone https://github.com/luoji/luoji.github.io.git
	进入 本地仓库目录：
	# cd luoji.github.io
	将 当前目录的仓库 推送到 GitHub 中的 Git 仓库
	# git push origin main

默认推送仓库时每次都需要手动输入密码

	查看远程仓库地址
	# git remote -v	#查看远程仓库地址
	origin	https://github.com/luoji/luoji.github.io.git (fetch)
	origin	https://github.com/luoji/luoji.github.io.git (push)

推送 push 测试

	# git push origin main
	Username for 'https://github.com': 
	Password for 'https://luoji@gmail.com@github.com': 
	#每次 push 都需要手动输入用户名和token

推送仓库不再提示用户名和密码的配置
原理：如果远程仓库地址的URL带有token，则执行 “git push origin main” 推送时不再提示需要输入用户名和密码：

	https://<your_token>@github.com/<USERNAME>/<REPO>.git

仓库设置为含有 token 的 远程仓库URL 命令（可以是协作的他人仓库）
：
	# git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git

解释说明范例：

	假设 Github 的 Clone 地址（HTTPS）是：
	https://github.com/luoji/luoji.github.io.git

	则带Token的地址格式为：
	https://<your_token>@github.com/luoji/luoji.github.io.git
	<your_token>：换成自己的token

	最终转换命令为：
	# git remote set-url origin https://ghp_b00H6@github.com/luoji/luoji.github.io.git

查看修改后的远程仓库地址

	# git remote -v
	origin  https://ghp_b00H6@github.com/luoji/luoji.github.io.git (fetch)
	origin  https://ghp_b00H6@github.com/luoji/luoji.github.io.git (push)

成功！此后执行推送命令 “git push origin main” 将不再提示需要输入 用户名 和 Token 密码!

> https://luoji.men/2022/09/how-to-cancel-manually-entering-login-password-every-time-in-github-push-warehouse-under-linux-environment/