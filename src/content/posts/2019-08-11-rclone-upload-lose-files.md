---
title: 解决 aria2 + rclone 自动上传文件丢失文件的问题
published: 2019-08-11
description: ''
image: ''
tags: [selfhost]
category: 'Note'
draft: false 
---
### 前言

最近和朋友合租了一台 VPS ，本来打算拿来刷 PT 站用，结果发现不太合适于是拿来做离线下载机了。因为我的专业有很多资料被各大垃圾公众号把持在手里在垃圾百度云上，于是我和朋友在干这么一件事情，我朋友负责获取各大公众号的资源，我就负责搬运到 Onedrive 上去，我现在终于有这个机会把这个过程自动化了。

### 发现问题

因为是合租的机器，怕把宿主机搞炸，于是选择使用 [moerats 大佬的 docker](https://www.moerats.com/archives/750/) 部署 aria2 ，rclone 也不难配置，在此不赘述。部署完成后就添加了 [moerats 大佬的自动上传文件脚本](https://www.moerats.com/archives/482)。这时候问题就来了，然后就发现无法在 docker 里安装 rclone 进行 rclone mount 。于是选择把 Onedrive 挂载在宿主机上，再把 Onedrive 目录挂载进 docker 里，下载完成后移动文件进这个目录就可以了。这时候开始发现问题了，在百度云下载文件夹的时候，总是会有几个文件无法上传到 Onedrive，于是另寻脚本。然后找到了同样是 [moerats 大佬提供的这个脚本](https://www.moerats.com/archives/697/) ，但是因为在 docker 里有权限问题，于是放弃，再寻。Google 时发现一个号称是 aria2 完美配置的 [repo](https://github.com/P3TERX/aria2_perfect_config) 还有这篇[博客文章](https://blog.codesofun.com/aria2-rclone-faq.html)，使用之，解决了无法在 docker 里挂载 Onedrive 的问题，但是发现还是有无法上传全部文件这个问题，我搜遍全网似乎没看到解决方式（也许是我这个垃圾搜索方式不对）。

### 解决问题

于是没办法，开始研究各位大佬的自动上传脚本，看到大佬们都使用了 Aria2 生成的代表文件数量的 `$2` 和文件路径的 `$3` 变量，作为一个基本看不懂 Linux shell 脚本的菜鸡，勉勉强强看得脑阔疼，胡乱猜测是 `$2` 文件数量传输错误或者 `$3` 下载完成时没被正确触发或者变量被覆盖导致无法上传。查阅 [Aria2官方手册](https://aria2.github.io/manual/en/html/aria2c.html) ，本来只是试图找一下到底是 aria2 背锅还是脚本背锅，然后我看到官方手册的 [Event Hook](https://aria2.github.io/manual/en/html/aria2c.html#event-hook) 一项中写着，在下载任务的开始，暂停，停止和完成都可以使用 `$2` 和 `$3` ，根据刚刚的猜想，如果比较晚下载的文件触发了脚本导致 `$2` 或者 `$3` 变量被覆盖，那么我在一开始下载就传递变量呢，最大下载任务被限制着，先下载完成的文件触发上传脚本直接就上传了，任务不会被结束，比较晚开始下载的文件也不怕覆盖前面的。于是查了下怎么跨脚本传参，发现这个想法完全可行，于是就修改了一下这份所谓[完美配置](https://github.com/P3TERX/aria2_perfect_config)：

1、新建 `filename.sh`，把 `autoupload.sh` 拆分成两份，该文件中 7-14 行均在处理文件路径，把该文件 7-14 行的内容复制到新建的 `filename.sh` 中并保存。

2、在 `autoupload.sh` 把 7-14 行和 `on-download-complete=/root/.aria2/delete.aria2.sh` 注释掉，因为此种方法不会建立上传 .aria2 后缀名文件的任务，所以可以注释，新增一行`source /path/to/filename.sh` 

【注：我在该文件中把 `rclone move -v` 改为 `rclone sync` 以防万一，不确定 `rclone move -v` 是否可行，尚未进行测试】

3、在 aria2.conf 中新增一行（原文件中此行被注释，取消注释补上 filename.sh 的路径也可以）

```on-download-start=/path/to/filename.sh```


至此，该问题终于被完美解决。对于本菜鸡来说绝对是个值得高兴的事情，第一次自己从头到尾完美解决一个问题，这种感觉就像游戏最终 boss 自己花了几个小时几天终于**无伤通关**了一样舒爽，好了，该睡觉了。