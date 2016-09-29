---
layout: post
title: 'Freeing Space On MySQL By Purging Old Binary Logs'
date: 2016-09-29 11:30:50 -0400
comments: true
categories:
---

I've been working a lot on testing and fleshing out online migrations on our hot MySQL tables lately.
In short, we have some tables that are very, very big (hundreds of GB) and we can't afford to incur
downtime to alter them. One solution for this that we're experimenting with is Github's [gh-ost](https://github.com/github/gh-ost).

In doing my experiments on staging I ran into some issues when the drive filled up.
The binary logs were truncated and couldn't be written to (no room) so there was potential
for data loss not only from the master not being able to accept writes but from potentially not
being able to write more to the logs.

Whether or not there was actual data loss was not a concern - I'd rather fix the problem by avoiding it altogether,
so going into production testing I wanted to be extra sure that we had the headroom to perform large migrations.
In general I'd recommend at least twice the size of the biggest table you want to migrate, preferably much more.
I checked the disk space and to my dismay the drives were almost full and I hadn't even begun much testing!

```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       40G  4.2G   34G  12% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev             15G   12K   15G   1% /dev
tmpfs           3.0G  436K  3.0G   1% /run
none            5.0M     0  5.0M   0% /run/lock
none             15G     0   15G   0% /run/shm
none            100M     0  100M   0% /run/user
/dev/xvde1      591G  580G   11G  99% /mnt/data
```

Well that's definitely not good. Luckily MySQL distributes data fairly conveniently on the drive, so you can go into the
directory and start taking measurements to diagnose the problem:

```sh
root@wistia-database-prod4:/mnt/data/mysql# ls -lh
total 299G
-rw-r----- 1 mysql mysql 416K Sep 28 20:49 aria_log.00000001
-rw-r----- 1 mysql mysql   52 Sep 28 20:49 aria_log_control
-rw-rw---- 1 mysql mysql 1.1G Sep 15 15:40 bin.010011
-rw-rw---- 1 mysql mysql 1.1G Sep 15 16:40 bin.010012
-rw-rw---- 1 mysql mysql 1.1G Sep 15 17:46 bin.010013
-rw-rw---- 1 mysql mysql 1.1G Sep 15 18:48 bin.010014
-rw-rw---- 1 mysql mysql 1.1G Sep 15 19:53 bin.010015
-rw-rw---- 1 mysql mysql 1.1G Sep 15 20:57 bin.010016
-rw-rw---- 1 mysql mysql 1.1G Sep 15 22:01 bin.010017
-rw-rw---- 1 mysql mysql 1.1G Sep 15 23:00 bin.010018
-rw-rw---- 1 mysql mysql 1.1G Sep 15 23:59 bin.010019
-rw-rw---- 1 mysql mysql 1.1G Sep 16 01:00 bin.010020
-rw-rw---- 1 mysql mysql 1.1G Sep 16 02:04 bin.010021
-rw-rw---- 1 mysql mysql 1.1G Sep 16 03:07 bin.010022
-rw-rw---- 1 mysql mysql 1.1G Sep 16 04:11 bin.010023
-rw-rw---- 1 mysql mysql 1.1G Sep 16 05:32 bin.010024
-rw-rw---- 1 mysql mysql 566M Sep 16 06:25 bin.010025
-rw-rw---- 1 mysql mysql 1.1G Sep 16 07:58 bin.010026
-rw-rw---- 1 mysql mysql 1.1G Sep 16 09:33 bin.010027
-rw-rw---- 1 mysql mysql 1.1G Sep 16 11:01 bin.010028
-rw-rw---- 1 mysql mysql 1.1G Sep 16 12:13 bin.010029
-rw-rw---- 1 mysql mysql 1.1G Sep 16 13:13 bin.010030
-rw-rw---- 1 mysql mysql 1.1G Sep 16 14:14 bin.010031
...

root@wistia-database-prod4:/mnt/data/mysql# ls -lh | grep 'bin\.0' | wc -l
302
```

You can see that the binary logs are about a gig a piece.
If we have 300 of them, well that's 50% of our drive capacity right there.

The binary logs are basically event logs.
They are primarily used in replication: the slave will connect to the server and start
tailing the logs and copy over any alterations to itself.
Thus, the old (already read) logs don't really have any purpose other than for backup safety.

So lets see where our slave has read up to. Running this on our slave:

```mysql
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
              Master_Log_File: bin.010311
          Read_Master_Log_Pos: 476112242
```

Here we see that our slave is pretty far along. This is saying the slave has read up to "bin.010311" and the given file offset.
Thus all those old logs (yes, the logs are numbered sequentially) can be thrown out.
(Important! I only have one slave in this case, but you should make sure that you make sure that *all* of your slaves
are past any logs you want to throw out).
Since I'd rather be superstitious than sorry, I'll leave a couple old logs more than I need to.
Running this on the master:

```mysql
MariaDB [(none)]> PURGE BINARY LOGS TO 'bin.010300';
```

Now if we check our usage:

```sh
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       40G  2.9G   35G   8% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev             15G   12K   15G   1% /dev
tmpfs           3.0G  432K  3.0G   1% /run
none            5.0M     0  5.0M   0% /run/lock
none             15G     0   15G   0% /run/shm
none            100M     0  100M   0% /run/user
/dev/xvde1      591G  253G  308G  46% /mnt/data
```

Much better! Do check [the MySQL documentation on PURGE BINARY LOGS](https://dev.mysql.com/doc/refman/5.7/en/purge-binary-logs.html) for yourself and
for further info.
