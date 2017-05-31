---
layout: post
title: "Restore HAProxy config from memory"
date: "2017-05-31 00:00:00 +0800"
---

## Why?

We have a DRBD setup that had a split brain situation, our loadbalancer server was running on
the server that we need to discard it's data, so after we fixed DRBD situation we noticed that
haproxy config file on disk was corrupted :S

haproxy was working fine because the way haproxy works it read the config file and import it into
the memory.

So we need to restore the config file from the memory, here is what we did:

<!--more-->

**NOTE**: If you're haproxy inside docker try to execute above commands inside docker container.

## Find HAProxy memory addresses

We need to find haproxy PID first
```
bash-4.3# ps axjf|grep haproxy
      6     249       6       6 ?             -1 S        0   0:00  \_ /usr/local/sbin/haproxy -p /run/haproxy.pid -f /usr/local/etc/haproxy/nl-haproxy.cfg -Ds -sf 248
    249     250     250     250 ?             -1 Ss       0   0:00      \_ /usr/local/sbin/haproxy -p /run/haproxy.pid -f /usr/local/etc/haproxy/nl-haproxy.cfg -Ds -sf 248
```

HAproxy uses **249** PID.
Then let's cat memory map of haproxy

```
bash-4.3# cat /proc/249/maps
7f0abbbf4000-7f0abc014000 rw-p 00000000 00:00 0
7f0abc014000-7f0abc26b000 r-xp 00000000 fd:01 394011                     /usr/lib/libpcre.so.1.2.6
7f0abc26b000-7f0abc26c000 r--p 00057000 fd:01 394011                     /usr/lib/libpcre.so.1.2.6
7f0abc26c000-7f0abc26d000 rw-p 00058000 fd:01 394011                     /usr/lib/libpcre.so.1.2.6
7f0abc26d000-7f0abc46e000 r-xp 00000000 fd:01 394013                     /usr/lib/libpcreposix.so.0.0.3
7f0abc46e000-7f0abc46f000 r--p 00001000 fd:01 394013                     /usr/lib/libpcreposix.so.0.0.3
7f0abc46f000-7f0abc470000 rw-p 00002000 fd:01 394013                     /usr/lib/libpcreposix.so.0.0.3
7f0abc470000-7f0abc862000 r-xp 00000000 fd:01 393376                     /lib/libcrypto.so.1.0.0
7f0abc862000-7f0abc87e000 r--p 001f2000 fd:01 393376                     /lib/libcrypto.so.1.0.0
7f0abc87e000-7f0abc889000 rw-p 0020e000 fd:01 393376                     /lib/libcrypto.so.1.0.0
7f0abc889000-7f0abc88d000 rw-p 00000000 00:00 0
7f0abc88d000-7f0abcaeb000 r-xp 00000000 fd:01 393377                     /lib/libssl.so.1.0.0
7f0abcaeb000-7f0abcaf0000 r--p 0005e000 fd:01 393377                     /lib/libssl.so.1.0.0
7f0abcaf0000-7f0abcaf6000 rw-p 00063000 fd:01 393377                     /lib/libssl.so.1.0.0
7f0abcaf6000-7f0abcd0a000 r-xp 00000000 fd:01 393379                     /lib/libz.so.1.2.8
7f0abcd0a000-7f0abcd0b000 r--p 00014000 fd:01 393379                     /lib/libz.so.1.2.8
7f0abcd0b000-7f0abcd0c000 rw-p 00015000 fd:01 393379                     /lib/libz.so.1.2.8
7f0abcd0c000-7f0abcd93000 r-xp 00000000 fd:01 393374                     /lib/ld-musl-x86_64.so.1
7f0abcf82000-7f0abcf92000 rwxp 00000000 00:00 0
7f0abcf92000-7f0abcf93000 r--s 00000000 fd:01 393326                     /etc/localtime
7f0abcf93000-7f0abcf94000 r--p 00087000 fd:01 393374                     /lib/ld-musl-x86_64.so.1
7f0abcf94000-7f0abcf95000 rw-p 00088000 fd:01 393374                     /lib/ld-musl-x86_64.so.1
7f0abcf95000-7f0abcf98000 rw-p 00000000 00:00 0
7f0abcf98000-7f0abd079000 r-xp 00000000 fd:01 394026                     /usr/local/sbin/haproxy
7f0abd279000-7f0abd27c000 r--p 000e1000 fd:01 394026                     /usr/local/sbin/haproxy
7f0abd27c000-7f0abd286000 rw-p 000e4000 fd:01 394026                     /usr/local/sbin/haproxy
7f0abd286000-7f0abd293000 rw-p 00000000 00:00 0
7f0abe0b2000-7f0abe26c000 rw-p 00000000 00:00 0                          [heap]
7fff0f12b000-7fff0f140000 rw-p 00000000 00:00 0                          [stack]
7fff0f1ff000-7fff0f200000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
```

We only need to dump the one that has write permission, ex: `rw-p`.
Memory address is the first column, ex: `7f0abbbf4000-7f0abc014000`

## dump memory

After we know the memory address that we need to dump, let's dump using *gdb* tool.

```
gdb --pid 249
GNU gdb (GDB) 7.10.1
Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-alpine-linux-musl".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word".
Attaching to process 249
Reading symbols from /usr/local/sbin/haproxy...done.
Reading symbols from /lib/libz.so.1...(no debugging symbols found)...done.
Reading symbols from /lib/libssl.so.1.0.0...(no debugging symbols found)...done.
Reading symbols from /lib/libcrypto.so.1.0.0...(no debugging symbols found)...done.
Reading symbols from /usr/lib/libpcreposix.so.0...(no debugging symbols found)...done.
Reading symbols from /usr/lib/libpcre.so.1...(no debugging symbols found)...done.
Reading symbols from /lib/ld-musl-x86_64.so.1...(no debugging symbols found)...done.
0x00007f0abcd61cb2 in ?? () from /lib/ld-musl-x86_64.so.1
(gdb)
```

Now we need to dump memory
```
(gdb) dump memory /tmp/7f0abc889000-7f0abc88d000 0x7f0abc889000 0x7f0abc88d000
```

## Automate gdb command

Because so many memory addresses and we don't know which one is the address that hold
haproxy configuration so we had to use this script:
https://gist.github.com/gabrielgrant/1235659

```
bash-4.3# python dumpmaps.py 249
dump memory 7f0abbbf4000 0x7f0abbbf4000 0x7f0abc014000
dump memory 7f0abc889000 0x7f0abc889000 0x7f0abc88d000
dump memory 7f0abcf82000 0x7f0abcf82000 0x7f0abcf92000
dump memory 7f0abcf95000 0x7f0abcf95000 0x7f0abcf98000
dump memory 7f0abd286000 0x7f0abd286000 0x7f0abd293000
dump memory 7f0abe0b2000 0x7f0abe0b2000 0x7f0abe26c000
dump memory 7fff0f12b000 0x7fff0f12b000 0x7fff0f140000
dump memory 7fff0f1ff000 0x7fff0f1ff000 0x7fff0f200000
dump memory ffffffffff600000 0xffffffffff600000 0xffffffffff601000


Run the commands above with `gdb --pid 249`
```

## Read the output file

Use `strings` command to read the binary file
```
strings 7f0abc889000
```
