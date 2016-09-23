---
layout: post
title: 'Connecting to MySQL Using SSH Tunnelling'
date: 2016-09-23 15:46:04 -0400
comments: true
categories:
---

My company has a tooling server which is one of a few machines that has direct firewall access to production.
That access is fairly limited though and blocks many ports (one of them being the ports we use for MySQL).

I recently wanted to set up [pt-heartbeat](https://www.percona.com/doc/percona-toolkit/2.2/pt-heartbeat.html) to monitor
replication lag for our MySQL instances. To do this a update `pt-heartbeat` process writes timestamped records to the
master database and then you can spin up monitor `pt-heartbeat` processes on the slaves to measure the lag in a black box
approach that winds up being more accurate than the built-in measurements provided by `SHOW SLAVE STATUS;`.

Let's consider a pair of servers, call them `master.db.com` and `slave.db.com`. The problem is from my current server I can
SSH into each, but I can't connect via the `mysql` client into each (which would indicate that `pt-heartbeat` would be able
to connect to each database).

The solution is to create an SSH tunnel:

```sh
ssh -f master.db.com -L 3306:master.db.com:3306 -N
ssh -f slave.db.com -L 3307:slave.db.com:3306 -N
```

`-f` will run SSH in the background, `-L` will forward traffic on my local port (e.g. `3307`) to the remote ip/port, and `-N` will
not run any commands on the remote machine (useful for port forwarding).

Once I've done that I can now connect to one of the databases with `mysql`:

```sh
mysql -h 127.0.0.1 -P 3307
#=> I'm in!
```
