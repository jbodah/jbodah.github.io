---
layout: post
title: 'Checking File Descriptor Progress in Running Processes'
date: 2018-07-11 11:16:24 -0400
comments: true
categories:
---

My coding life over the past few months has been less about coding systems and features and more about shepherding data
and writing automation.
I guess you could call it dev ops.
Anyways, I've been doing *a lot* of scripting.

Working directly with the OS is a lot different than working in the framework of a formal programming language.
I've found myself running into problems that I normally have simple answers to.
I've had to learn how to deal with coarse level performance debugging, how to implement process-level synchronization, and just
general monitoring to figure out what my processes are doing.

One cool trick I just learned was that you can check the position of a file descriptor for an open process using `/proc`.
`/proc` and `/dev` are two really neat features Linux provides that give you all sorts of information about the running processes (the
info here is what programs like `top` show you).

Let's motivate this with a real life example.
I have a simple process along the lines of `cat my_file.log | do_some_processing`.
I can verify that it is working with `strace -p 1234` where `1234` is the pid.
When I do this I don't see any blocking on reads/writes, so the data is flowing.

The next question is "how fast is it?" or "what's my progress?".
We can use `/proc/<PROCESS>/fdinfo/<FILE_DESCRIPTOR>` to check:

```
$ # I have a pipeline running which is processing a log file
$ # It looks like `cat workspace/archiver-ip-10-0-100-44-000000.2017-12-29_23.log.gz.out.pass | do_something`

$ # get pid
$ ps aux | grep cat
ec2-user 10810  0.1  0.0 107980   752 pts/0    S+   14:39   0:01 cat workspace/archiver-ip-10-0-100-44-000000.2017-12-29_23.log.gz.out.pass
ec2-user 10992  0.0  0.0 110512  2072 pts/1    R+   15:06   0:00 grep --color=auto cat

$ # check open file descriptors; 0-2 are stdin, stdout, stderr so 3 must be my open file
$ cat /proc/10810/fdinfo/
0  1  2  3
$ cat /proc/10810/fdinfo/3
pos:	1846607872
flags:	0100000
mnt_id:	23

$ # now check how big my file is
$ ls -l workspace/archiver-ip-10-0-100-44-000000.2017-12-29_23.log.gz.out.pass
-rw-rw-r-- 1 ec2-user ec2-user 3127457998 Jul 11 14:38 workspace/archiver-ip-10-0-100-44-000000.2017-12-29_23.log.gz.out.pass

$ # and figure out what percent my fd is through my file
$ ruby -e 'puts 1846607872.0 / 3127457998'
0.5904500949911718
```

Sweet, so I'm about 60% of the way through my file. I can now use that to decide whether it's worth trying to optimize my processing code further
and whether it's ready to be parallelized.
