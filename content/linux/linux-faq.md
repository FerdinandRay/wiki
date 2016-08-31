---
title: Linux FAQ
date: 2016-07-29 18:07
update: 2016-08-31 12:49
---

***1. GPG签名验证失败：由于没有公钥，下列签名无法进行验证***

```shell
sudo apt-key adv --recv-keys --keyserver keyserver.Ubuntu.com DDA4DB69  
```

***2. 移动硬盘无法安全移除，ubuntu下安全卸载后又自动挂载***

```shell
// 1. 需要安装udisks命令
sudo apt-get install udisks
// 2. 卸载硬盘
sudo umount /dev/sdb1(这里填自己的移动硬盘)
// 3. 关闭电源
sudo udisks --detach /dev/sdb
```

***3. Android-Studio tool.jar seems not什么***

	环境：ubuntu 14.04 amd64

	原因：安装了OpenJdk，通过java -version和javac -version如果版本不同，且前者出现了OpenJdk字样。

	方法：命令行删除暂时不会，老说找不到，正则了也不行，所以图形界面直接找到删除了就行

	后续：还可能会有一个错误，看下面。

***4. Android-Studio No JDK Found***

	原因：因为之前删除了OpenJdk，虽然你的环境变量中有了jdk的配置，但还是不能打开Android-Studio

	方法：执行该命令`sudo update-alternatives --install /usr/bin/java java $JAVA_HOME/bin/java 100`

	后续：不知道为什么，单纯网上找的答案。
