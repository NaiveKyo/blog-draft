---
title: Linux Basic Knowledge Review (一)
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110644.jpg'
coverImg: /img/20220425110644.jpg
cover: false
toc: true
mathjax: false
date: 2022-12-17 23:44:30
summary: "Linux 基础知识回顾 (一)"
categories: "Linux"
keywords: "Linux"
tags: "Linux"
---

# 参考

- 书籍：鸟哥的 Linux 私房菜：基础学习篇（第四版）

- https://refspecs.linuxfoundation.org/



# FHS 2

参考：https://refspecs.linuxfoundation.org/fhs.shtml

暂时先介绍 FHS[2] 规范：

FHS 根据文件系统使用的频繁与否和是否允许使用者随意动用，将目录定义为四种交互形态：

|                      | 可分享的（shareable）       | 不可分享的（unshareable） |
| -------------------- | --------------------------- | ------------------------- |
| 不变的（static）     | /usr（软件放置处）          | /etc（配置文件）          |
|                      | /opt（第三方软件）          | /boot（开机与核心文件）   |
| 可变动的（variable） | /var/mail（使用者邮件信箱） | /var/run（程序相关）      |
|                      | /var/spool/news（新闻群组） | /var/lock（程序相关）     |

简要了解一下四种类型：

- 可分享的：可以分享给其他系统挂载的目录，包括可执行文件与使用者的邮件等数据，是能够分享给网络上其他主机挂载用的目录；
- 不可分享的：自己机器上运行的配置文件或者与程序有关的 socket 文件等等，由于仅与自身机器有关，所以不适合分享给网络上的其他主机；
- 不变的：有些数据是不会经常变动的，即使是不同的 Dirstribution 这些也不会改变。比如函数库、文件说明文档、系统管理员所管理的主机服务配置文件等等；
- 可变动的：经常改变的数据，例如登录文件、一般用户可自行接收的新闻群组等等。

事实上，FHS 针对目录树架构仅定义出三层目录下面应该放置什么数据而已，分别是下面三个目录：

- / （root，根目录）：与开机系统有关；
- /usr（unix software resource）：与软件安装/执行有关；
- /var（variable）：与系统运行过程有关。

## 重点：根目录

重点：/ 根目录的意义与内容

根目录是整个系统中最重要的一个目录，因为不但所有的目录都是由根目录衍生出来的，同时根目录也与开机/还原/系统修复等动作有关。由于系统开机需要特定的开机软件、核心文件、开机所需程序、函数库等等文件数据，若系统出现错误，根目录也需要包含有能够修复文件系统的程序。

因为根目录是那么的重要，所以在 FHS 要求根目录不要放在非常大的分区内，因为越大的分区你就会放入越多的数据，如此一来根目录所在的分区发生错误的几率就会越大。

因此 FHS 标准建议：根目录（/）所在分区应该越小越好，且应用程序所安装的软件最好不要与根目录放在同一个分区内，保持根目录越小越好。如此不但性能较佳，且根目录所在的文件系统发生错误的概率也较低。

FHS 定义在根目录（/）下面应该要有一下次要目录，即使没有实体目录，FHS 也希望至少有链接文件存在：

### 必须存在的目录

| 目录   | 放置的文件内容                                               |
| ------ | ------------------------------------------------------------ |
| /bin   | 系统有很多可执行文件的目录，但 /bin 比较特殊。因为它放置的是在单人维护模式下还能够被操作的命令。在 /bin 下面的指令可以被 root 与一般账户所使用，主要有：cat、chmod、chown、date、mv、mkdir、cp、bash 等等常用的指令。 |
| /boot  | 这个目录主要放置开机会使用的文件，包括 Linux 核心文件以及开机所需要的配置文件等等。Linux kernel 常用的文件名为：vmlinuz，如果使用的是 grub2 这个开机管理程序，则还会存在 /boot/grub2/ 这个目录。 |
| /dev   | 在 Linux 系统上，任何设备与周边设备都是以文件的形式存在于 /dev 目录中的。只需要读取这个目录下的某个文件，就等于存取某个设备。比较重要的文件有 /dev/null、/dev/zero、/dev/tty、/dev/loop、/dev/sd 等等。 |
| /etc   | 系统主要的配置文件几乎都放置在这个目录下，例如人员的账户密码文件、各种服务的启动文件等等。一般来说，这个目录下的各文件属性是可以让一般使用者查阅的，但是只有 root 有权利修改。FHS 建议不要放置可执行文件（binary）在这个目录。比较重要的文件有：/etc/modprobe.d/、/etc/passwd、/etc/fstabl、/etc/issue 等等。另外，FHS 还规定几个重要的目录最好要存在 /etc/ 下面：<br/>/etc/opt（必要）：这个目录放置第三方软件（/opt 下面）的相关配置文件；<br/>/etc/X11（建议）：与  X Window 有关的各种配置文件都在这里，尤其是 xorg.conf 这个配置文件；<br/>/etc/sgml（建议）：与 SGML 格式有关的各项配置文件；<br/>/etc/xml/（建议）：与 XML 格式有关的各项配置文件。 |
| /lib   | 系统的函数库非常多，而 /lib 放置的则是开机时会用到的函数库，以及在 /bin 或 /sbin 下面的指令会调用的函数库。另外，FHS 要求下面的目录必须存在 /lib/modules/：这个目录主要放置可抽换式的核心相关模块（驱动程序）。 |
| /media | media 是 "媒体" 的意思，顾名思义，这里目录下面放置的就算可移除的设备，包括软盘、光盘、DVD 等设备都暂时挂载于此。常见的文件名有：/media/floppy、/media/cdrom 等等。 |
| /mnt   | 如果想要暂时挂载某些额外的设备，一般建议你可以放在这个目录中。在早期的时候，这个目录的用途和 /media 一样，只是有了 /media 后，这个目录暂时就用来挂载了。 |
| /opt   | 这个是给第三方软件放置的目录。什么是第三方软件呢？e.g. KDE 这个桌面管理系统是一个独立的计划，但是它可以安装到 Linux 系统中，因为 KDE 的软件就是建议放置到此目录下的。另外，如果你想要自行安装额外的软件（非原本的 distribution 提供的），那么也可以将软件安装在 /opt 目录下。不过，以前的 Linux 系统中，我们还是习惯于放在 /usr/local 目录下。 |
| /run   | 早期的 FHS 规定系统开机后产生的各项信息应该放在 /var/run 目录下，新版的 FHS 则规范到 /run 下面。由于 /run 可以使用内存来模拟磁盘，因此性能上会好很多。 |
| /sbin  | Linux 有非常多的指令用来设置系统环境，这些指令只有 root 才能用来 "设置" 系统，其他使用者最多只能用来 "查询" 而已。放在 /sbin 下面的为开机过程中所需要的，里面包含了开机、修复、还原系统所需要的指令。至于某些服务器软件程序，一般则放置在 /usr/sbin/ 当中。 |
| /srv   | srv 可以视为 "service" 的缩写，是一些网络服务启动后，这些服务所需要取用的数据目录。常见的服务如 WWW、FTP 等等。举例来说，WWW 服务器需要的网页数据就可以放置在 /srv/www/ 里面。不过，系统的服务数据如果尚未要提供给实际网络中的任何人浏览的话，默认还是建议放置到 /var/lib 下面。 |
| /tmp   | 这是让一般使用者或者是正在执行的程序暂时放置文件的地方。这个目录是任何人都能存取的，所以需要定期清理一下。当然，重要数据不能放在这里，因为 FHS 甚至建议在开始的时候应该将 /tmp 中的所有数据清除。 |
| /usr   | 见下文                                                       |
| /var   | 见下文                                                       |

### 可以存在的目录

| 目录   | 放置的文件内容                                               |
| ------ | ------------------------------------------------------------ |
| /home  | 系统默认的使用者主文件夹（home directory）。当我们新增一个使用者账户时，默认的使用者主文件夹就是这里。比较重要的是主文件夹有两种代号：<br/>`~`：代表当前使用者的主文件夹；<br/>`~dmtsai`：~ 后面跟指定用户名，表示指定用户的主文件夹，例子中就是 dmtsai 的主文件夹。 |
| /lib64 | 用来存在与 /lib 不同格式的二进制函数库，例如支持 64 位的 /lib64 函数库等等 |
| /root  | 系统管理员（root）的主文件夹。之所以放在这里，是因为如果进行单人维护模式且仅挂载根目录时，该目录就能拥有 root 的主文件夹，所以我们会希望 root 的主文件夹与根目录放置在同一个分区中。 |

### 额外补充

除了 FHS 针对根目录所定义的标准，Linux 还有一些目录可以了解一下：

| 目录        | 应放置文件内容                                               |
| ----------- | ------------------------------------------------------------ |
| /lost+found | 这个目录是使用标准的 ext2/ext3/ext4 文件系统格式才会产生的一个目录，目的在于当文件系统发生错误时，将一些遗失的片段放置在该目录下。不过如果使用的是 xfs 文件系统就不会存在这个目录。 |
| /proc       | 这个目录本身是一个 "虚拟文件系统（virtual filesystem）"，它放置的数据都是在内存当中，例如系统核心、进程信息（process）、周边设备的状态以及网络状态等等。因为这个目录下的数据都是在内存当中的，所以本身不占用任何磁盘空间，比较重要的文件例如：/proc/cpuinof、/proc/dma、/proc/interrupts、/proc/ioports、/proc/net/* 等等 |
| /sys        | 这个目录其实和 /proc 非常类似，也是一个虚拟文件系统，主要也是记录核心与系统硬件相关的信息。包括目前已载入的核心模块与核心侦测到的硬件设备信息等等。这个目录同样不占用磁盘空间。 |

早期 Linux 在设计的时候，若发生问题时，救援模式通常仅挂载根目录而已，因此有五个重要的目录被要求一定要与根目录放置在一起， 那就是 `/etc, /bin, /dev, /lib, /sbin` 这五个重要目录。现在许多的 Linux distributions 由于已经将许多非必要的文件移出 /usr 之外了， 所以 /usr 也是越来越精简，同时因为 /usr 被建议为 “即使挂载成为只读，系统还是可以正常运行” 的模样，所以救援模式也能同时挂载 /usr ，例如我们的这个 CentOS 7.x 版本在救援模式的情况下就是这样。因此那个五大目录的限制已经被打破了，例如CentOS 7.x 就已经将 /sbin, /bin, /lib 通通移动到 /usr 下面了。



## /usr 目录

/usr 的目录的意义与内容：

根据 FHS 定义，/usr 里放置的数据属于可分享的与不可变动的（shareable、static），在通过网络进行分区的挂载时，/usr 确实可以分享给区域网络的其他主机来使用。

很多人会误解 /usr 是 user 是缩写，其实 usr 是 Unix Software Resource 的缩写，即 "Unix 操作系统软件资源" 放置的目录，而不是使用者的数据。这点要注意，FHS 建议所有软件开发者，应该将他们的数据合理的分配放置到这个目录下面的次目录中，而不要自行创建该软件独立的目录。

因为是所有系统（包括不同的 distribution）默认的软件都会放在 /usr 里面，因此这个目录有点类似 Windows 系统的 "`C:\Windows\`"（当中的一部分）+ "`C:\Program Files\`" 这两个目录的综合体，系统刚安装完毕，这个目录占用最多的硬盘容量，一般来说，/usr 建议的次目录有下面这些：

### 必须存在的目录

| 目录        | 应放置文件内容                                               |
| ----------- | ------------------------------------------------------------ |
| /usr/bin/   | 所有用户能够使用的命令都放在这里，目前 CentOS 7 已经将全部的使用者指令放置于此，而使用链接文件的方式将 /bin 连接至此。也就是说，/usr/bin 和 /bin 是一模一样的，另外，FHS 要求该目录不应该有子目录。 |
| /usr/lib/   | 和 /lib 的功能基本一致，且 /lib 就是链接到这里。             |
| /usr/local/ | 系统管理员在本机自行安装自己下载的软件（非 distribution 默认提供），建议安装到此目录下，这样便于管理。 |
| /usr/sbin/  | 非系统正常运行所需要的指令。最常见的就是某些网络服务器软件的服务指令（daemon）。基本功能和 /sbin 差不多，目前 /sbin 已链接至此。 |
| /usr/share/ | 主要放置只读架构的数据文件，当然也包括共享文件。在这个目录下放置的数据是几乎不分硬件架构均可读取的数据，因为几乎都是文件文件 |

### 可以存在的目录

| 目录          | 应放置的文件内容                                             |
| ------------- | ------------------------------------------------------------ |
| /usr/games/   | 与游戏相关的数据放置处。                                     |
| /usr/include/ | c/c++ 等程序语言的文件开始（header）与包含档（include）放置处，当我们以 tarball 方式（*.tar.gz 的方式安装软件）安装某些数据时，会使用该目录下面的很多包含档。 |
| /usr/libexec/ | 某些不被一般使用者惯用的可执行文件或者脚本（script）等等，都会放置到此目录下。 |
| /usr/lib64/   | 与 /lib64 功能相同，目前 /lib64 链接至此。                   |
| /usr/src/     | 一般源代码建议放置到这里，src 有 source 的意思。至于核心源代码则建议放置在 /usr/src/linux/ 目录下。 |

## /var 目录

/var 的意义和内容：

如果 /usr 是安装是会占用较大磁盘容量的目录，那么 /var 就是在系统运行后才会渐渐占用磁盘容量的目录。

因为 /var 目录主要针对常态性变动的文件，包括高速缓存（cache）、登录文件（log file）以及某些软件运行时所产生的文件，包括程序文件（lock file，run file），或者例如 MySQL 数据库的文件等等。

常见的次目录如下所示。

### 必须存在的目录

| 目录        | 应放置文件内容                                               |
| ----------- | ------------------------------------------------------------ |
| /var/cache/ | 应用程序本身运行过程会产生的一些中间文件。                   |
| /var/lib/   | 程序本身执行过程中，需要使用到的数据文件所在的目录。在此目录下各自的软件应该要有各自的目录。比如，MySQL 的数据库放置到 /var/lib/mysql ，而 rpm 的数据库则放到 /var/lib/rpm 中。 |
| /var/lock/  | 某些设备或者文件资源一次只能被一个应用程序所使用，如果同时有两个程序使用该设备，就可能会产生错误，因此就需要将该设备上锁（lock），以确保该设备只会给单一软件使用。目前该目录已被移到 /run/lock 中。 |
| /var/log/   | 非常重要的目录，这里是登录文件放置的目录，里面比较重要的文件如 /var/log/messages、/var/log/wtmp（记录登录者的信息）等等。 |
| /var/mail/  | 放置个人电子邮件信箱的目录，不过这个目录也已经放置到 /var/spool/mail 目录中，通常这两个目录互为链接文件。 |
| /var/run/   | 某些程序或者服务启动后，会将他们的 PID 放置在这个目录下，与 /run 目录相同，这个目录链接到 /run 了。 |
| /var/spool/ | 这个目录通常放置一些伫列数据，所谓的 "伫列" 就是排队等待其他程序使用的数据，这些数据被使用后通常都会被删除。例如，系统收到新的邮件会放到 /var/spool/mail 中，一旦使用者收下该信件，则该信件原则上将会立即被删除。信件如果暂时寄不出去就会被放到 /var/spool/mqueue/ 中等到送出去后就被删除。如果是工作调度数据（crontab），就会被放置到 /var/spool/cron/ 目录中。 |

## 总结

简单了解了 FHS 后，推荐阅读官方英文版本，现在最新的是 FHS 3.0，了解这些有利于我们更加深入学习 Linux 的知识。