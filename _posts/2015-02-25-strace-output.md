---
layout: post
title:  "Strace output"
date:   2015-02-25 00:00:00
categories: strace, tunning
---

Every Sysadmin knows about strace command and we're using it in daily basis, but never come to me once to redirect the output to different file for more analyzing "you know strace output it's tremendously fast", so I thought why not use bash redirection

```
$ sudo strace -p 4148 > /tmp/strace.out
```

But it didn't work still output appeared in my screen, checking strace man page will fine the answer which is use `-o` option, for example 

```
$ sudo strace -p 4148 -o /tmp/strace.out 
Process 4148 attached
```

checking around finding out that when you run the strace command it'll run another strace process as child of it, so apparently the second process is the one who send the output to our stdout, so the only option is to use `-o` option.

```
 5110  5301  5301  5110 pts/4     5850 S     1000   0:00      \_ bash
 5301  5850  5850  5110 pts/4     5850 S+       0   0:00          \_ sudo strace -p 4148 -o /tmp/strace.out
 5850  5851  5850  5110 pts/4     5850 S+       0   0:00              \_ strace -p 4148 -o /tmp/strace.out
```

MhdAli
