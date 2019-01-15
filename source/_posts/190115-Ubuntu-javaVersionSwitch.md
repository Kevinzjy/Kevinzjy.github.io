---
title: Ubuntu实现Java版本切换
date: 2019-01-15 16:47:15
categories: Ubuntu
tags: [Ubuntu, ]
---

在使用IGV的时候，发现软件提示需要 java8，然而系统上已经同时安装了 java8 和 java11

```bash
sudo apt install openjdk-8-jdk
sudo apt isntall openjdk-11-jdk
```

可以看到 `/usr/bin/java` 指向的是 java11 的版本，因此我们只需要重新设置成 java8 就好了

```bash
$ java -version
openjdk version "10.0.2" 2018-07-17
OpenJDK Runtime Environment (build 10.0.2+13-Ubuntu-1ubuntu0.18.04.4)
OpenJDK 64-Bit Server VM (build 10.0.2+13-Ubuntu-1ubuntu0.18.04.4, mixed mode)
```

将系统默认使用的 java 版本更改为 java8

```bash
$ sudo update-alternatives --config java

There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      300       manual mode
* 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

输入对应的编号之后，就可以成功使用需要的 jdk 版本了

```bash
$ java -version
openjdk version "1.8.0_191"
OpenJDK Runtime Environment (build 1.8.0_191-8u191-b12-0ubuntu0.18.04.1-b12)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```