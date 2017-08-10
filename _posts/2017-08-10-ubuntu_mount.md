---
layout: post
title: Mount Protocol Error
categories: [Ubuntu]
keywords: Ubuntu, Mount
---

使用Vbox装了Ubuntu 10.04系统，使用Shared folder，在用下面的命令Mount时始终出现Protocol Error：

sudo mount -t vboxsf -o rw,uid=1000,gid=1000 chenshun ~/host/

这条命令本身没有错误。使用下面的命令会报告某些文件系统需要/sbin/mount.{type}文件，{type}表示文件系统名称。

sudo mount -t vboxsf chenshun ~/host/

检查了/sbin/mount.vboxsf后，发现错误发生的原因是/sbin/mount.vboxsf symlink指向的文件不存在，因此通过find的命令查找mount.vboxsf文件：

find / -name "mount.vboxsf"

找到文件/opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions/mount.vboxsf，然后用ln命令覆盖掉/sbin/mount.vboxsf。

ln -f -t /sbin/ /opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions/mount.vboxsf

然后重新运行第一条命令，挂载成功。

   