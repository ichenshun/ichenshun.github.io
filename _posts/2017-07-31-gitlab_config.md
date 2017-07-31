---
layout: post
title: GitLab配置手册
categories: [GitLab]
keywords: GitLab, 配置
---

1. 如何修改仓库clone url
   1. sudo vim /etc/gitlab/gitlab.rb ，将external_url修改成对应的地址
   2. sudo gitlab-ctl reconfigure