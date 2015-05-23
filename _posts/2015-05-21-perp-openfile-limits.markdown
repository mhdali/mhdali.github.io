---
layout: post
title: "perp openfile limits"
published: true
---

# Perp with open file limits

If you're using perp to control your processes instead of `systemd`, `upstart`, `xinit` ...etc, you may hit open file limitation issue, today post is to descripe more deep into this issue and how to solve it.

##Explaination

`CentOS 6.x` using `upstart` as it's init daemon, so whenever you install `perp` you need to configure it to work under `upstart`. check [] for more info how to implement perp with upstart.

However after we implement our `perp` setup and configure it to control our service by using `runtool` command to pass the service arguments and options, in this case the service is `logstash` with emabaded `elasticsearch`, after we run it with perp we start facing `too many opened files` issue.

Our system already has high number of big files configured on it on `/etc/security/limits.conf` file, by these two lines:

```
root soft nofile 48000
root hard nofile 48000
```

BUT the problem still appears in `logstash` logs. 

Checking process limits file show different figure than the one mentioned in `limits.conf` file:


```console
# cat /proc/$(pgrep java)/limits | grep "open files"
Max open files            4096               4096               files
```
##Solving

###runtool

Checking `runtool` manual pages to find about `-R` option, quoting man pages:

```man
-R rlims
  runlimit(8).  Sets up the soft resource limits for program where rlims is given as a single contiguous string in the form:

    <rlim> = <value> [: ...]

  Where:

  rlim:  single character in the set ‘a’, ‘c’, ‘d’, ‘f’, ‘m’, ‘o’, ‘p’, ‘r’, ‘s’, or ‘t’.

  ‘=’:   the literal character ‘=’.

  value: A  positive  numeric  value for the resource limit, or the special character ‘!’ used to specify ‘‘unlimited’’, that is, sets the soft limit equal to the
         current hard limit.
```

We configure `runtool` with `-R o:999999` option to increase open files limit to it's maximum.

BUT after doing so noting changed, `too many opened files` still in there, and process `limits` file still having `4096` limits of open files.

```console
# cat /proc/$(pgrep java)/limits | grep "open files"
Max open files            4096               4096               files
```

###upstart

Since `Upstart` is controlling perp it could be open file limitation coming from it, adding the following line to our init `perp.conf` file SOLVE our issue.

```
limit nofile 999999 999999
```

Now checking logstash process limits gives us.

```console
# cat /proc/$(pgrep java)/limits | grep "open files"
Max open files            999999               999999               files
```

Finally no more `too many opened files` issue.
