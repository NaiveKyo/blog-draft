# Kafka Monitor Tool

有了 Kafka 环境，自然需要一个方便的监控工具用于管理 Kakfa。

参考文章：

- https://stackoverflow.com/questions/49276785/monitoring-ui-for-apache-kafka-kafka-manager-vs-kafka-monitor

本文选择较为传统的 Kafka Manager 作为监控工具，它的最新版支持 Kafka 3.0+ 但是需要 Java 11+，所以我们选择 2.0+ 版本适配本地 Java 8 环境；

地址：

- https://github.com/yahoo/CMAK
- https://github.com/yahoo/CMAK/tree/2.0.0.2

更多详细特性和配置请参考 Github 上的说明；

# 

# CMAK（Kafka Manager）

CMKA（Cluster Manager for Apache Kafka）之前也叫做 Kafka Manager

## 替换  yum 镜像

由于 CentOS 默认使用的 yum 镜像比较慢，我们可以替换为国内的 aliyum 镜像。

参考：https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11bkZfGC

```bash
# 首先备份默认的 yum 源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 然后下载 aliyum 镜像作为默认镜像源
# CentOS8 (centos8官方源已下线，建议切换centos-vault源)
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

# CentOS7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 清空缓存
yum clean all
# 生成缓存
yum makecache
```

非阿里云 ECS 用户会出现 Couldn't resolve host 'mirrors.cloud.aliyuncs.com' 信息，不影响使用。用户也可自行修改相关配置: eg:

```bash
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

## Install sbt

由于 CMAK 是基于 Scala 编写的，要安装该软件就需要使用 Scala 的编译工具 sbt。

sbt 官网：https://www.scala-sbt.org/index.html

根据自身环境在官方文档中找到对应的安装方式，本文系统环境是 CentOS 7：

```bash
# remove old Bintray repo file
sudo rm -f /etc/yum.repos.d/bintray-rpm.repo || true
curl -L https://www.scala-sbt.org/sbt-rpm.repo > sbt-rpm.repo
sudo mv sbt-rpm.repo /etc/yum.repos.d/
sudo yum install sbt
```

然后随便输入 sbt 的某个命令：

```bash
# 查看版本号
sbt -V
# 或者
sbt --version
```

此时会出现卡很久的情况，这是在下载相关的依赖包，需要耐心等待，因为 sbt 默认使用的是 maven 的 repository，有些可能会被墙。

```bash
downloading sbt launcher 1.8.0
[info] [launcher] getting org.scala-sbt sbt 1.8.0  (this may take some time)...
[info] [launcher] getting Scala 2.12.17 (for sbt)...
sbt version in this project: 1.8.0
sbt script version: 1.8.0
```



## Complie CMAK

<font color='red'>最后没有编译成功，所以暂时不推荐使用该监控工具。</font>

使用 CMAK（Cluster Manager Of Apache Kafka，以前叫做 Kafka Manager）2.0.0.2 需要有以下环境：

- Java 8+；
- [sbt](https://www.scala-sbt.org/)：a build tool for Scala, Java, and [more](https://github.com/d40cht/sbt-cpp). It requires Java 1.8 or later.

CentOS7 安装 JDK 环境可以参考之前的[文章](https://naivekyo.github.io/2021/07/06/centos7-install-jdk/)，这里就不再赘述了。

从 Github 上下载所需的 CMAK 版本，解压后进入 CMAK 目录，开始构建并部署项目：

```bash
[root@localhost CMAK-2.0.0.2]# ./sbt clean dist
Downloading sbt launcher for 1.2.8:
  From  http://repo.scala-sbt.org/scalasbt/maven-releases/org/scala-sbt/sbt-launch/1.2.8/sbt-launch.jar
    To  /root/.sbt/launchers/1.2.8/sbt-launch.jar
Download failed. Obtain the jar manually and place it at /root/.sbt/launchers/1.2.8/sbt-launch.jar
```

第一次也许会出现这种情况，前面安装的 sbt 版本和 CMAK 所对应的 sbt 版本也许不一致，此时可以到[官网](https://www.scala-sbt.org/download.html)去下载指定版本的 sbt 的压缩包，解压后在 bin 目录找到所需的 jar 包，放到提示信息中指定的位置。

完成上述步骤后再次输入命令：

```bash
[root@localhost CMAK-2.0.0.2]# ./sbt clean dist
Getting org.scala-sbt sbt 1.2.8  (this may take some time)...
downloading https://repo1.maven.org/maven2/org/scala-sbt/sbt/1.2.8/sbt-1.2.8.jar ...
        [SUCCESSFUL ] org.scala-sbt#sbt;1.2.8!sbt.jar (3051ms)
downloading https://repo1.maven.org/maven2/org/scala-lang/scala-library/2.12.7/scala-library-2.12.7.jar ...
# long long time ...
```

这里要等待很长时间，包括下载各种 jar 包以及更新项目引用。