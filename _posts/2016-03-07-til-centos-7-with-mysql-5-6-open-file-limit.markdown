---
layout: post
title: "[TIL] Centos 7 with MySQL 5.6 open file limit"
date: "2016-03-07 16:47:31 +0800"
---

## What?
Today I tried to upgrade MySQL 5.6.27 to 5.6.29 on one of our database servers, that running on CentOS version 7.

However after I upgraded the package I started seeing this error many times:

```
[ERROR] /usr/sbin/mysqld: Can't open file: '<TABLE_NAME>' (errno: 24 - Too many open files)
```
<!--more-->

## Why?
So even if you go and extend the limit on `/etc/security/limits.conf` like this one below **it wont work**

```
mysql  soft  nofile  49152
mysql  hard  nofile  65536
```

Because CentOS 7 using systemd that came to me that it cause this issue, because systemd control open files as well!!



## How to fix?

So I went to `/etc/systemd/system/mysql.service` and added this line

```
LimitNOFILE=40000
```

Then refresh systemd with

```
systemctl daemon-reload
```

and make sure also MySQL restarted as well

```
systemctl restart mysqld
```

## Final tip

This issue should sorted by MySQL team because the default package doesn't come with a bigger number of open files.
