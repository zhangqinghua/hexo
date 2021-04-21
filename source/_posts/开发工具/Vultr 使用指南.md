---
title: Vultr 使用技巧

categories:
- 开发工具

date: 2021-01-10
---

## 重制密码
在我们使用Vultr VPS的时候，可能在某些情况下需要修改服务器的root密码，比如当VPS的快照恢复后原root密码会过期，这时需要重置root密码才能进行连接服务器，或者某些系统脚本也有可能会导致root密码无效，以及粗心大意自己忘记了Vultr VPS的root密码。

下面的教程将教大家怎样重置Vultr 的root密码。但是Vultr的不同操作系统修改root密码的方法是不一样的。本文将以CentOS 6 / CentOS 7系统为例，教大家如何修改Vultr 的root密码。

一、进入Vultr后台管理面板

按照下图指示，打开在线 Console，点击“View Console”。

![](001.png)

二、重启VPS

点击右上角的“Send CtrlAltDel”如图，重新启动VPS。

![](002.png)

三、编辑启动命令

当界面出现上图后，然后编辑第一启动项（按 e 键进入编辑）

![](003.png)

然后按键盘上的 “下方向键” 向下移动光标，找到“linux16” 开头这行，将：

```
ro
```
改为：

```
rw init=/sysroot/bin/sh
```

![](004.png)

四、重置密码

当按照上面修改好后，按 Ctrl + X 启动单用户模式。输入：

```
chroot /sysroot
```

回车进入系统，输入下面命令进行重置密码：

```
passwd
```

五、重启服务器

修改好密码后，输入下面指令进行重启：

```
reboot -f
```

大功告成！！！