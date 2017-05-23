---
layout: post
title: 'Learning About Kubernetes and UNIX Signals'
date: 2017-05-23 13:49:44 -0400
comments: true
categories:
---

In this post we're going to learn how to play with Kubernetes locally.
At Wistia we've been using Kubernetes in production for several months
now and it has proven to be a really slick and impressive container
orchestration service.

At a high level, Kubernetes lets you deploy Docker containers for your applciation
(which are called "pods" in Kubernetes). Kubernetes handles a lot of the messy details
of doing this for you like implementing rollbacks, blue-green deployments, log aggregation,
host filtering, secrets management, and a whole lot more.

To guide this tour, I'm going to take an actual problem that I'm trying to
solve at work. We host a cluster of HTTP services in Kubernetes (in this case Elixir services).
These services maintain a processing pipeline that runs relatively long and expensive jobs
that we want to allow to drain whenever we deploy. That way, they won't just get torn down
immediately when the deploy happens.

When Kubernetes decides it no longer needs a pod, it will tear down each container of the pod with a SIGTERM ([source](https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods)).
This is followed by a grace period and then a SIGKILL to any containers that is still up.
By default, Elixir apps will exit immediately when they receive a SIGTERM.
I've discussed our approach to handling SIGTERM in Elixir more gracefully in [another blog post](/blog/2017/05/18/implementing-graceful-exits-elixir/), so for now you can assume
that our service already is handling SIGTERM properly.
The open question we have is whether or not the signal is actually getting from Kubernetes to the correct process.

First we'll have to [install Kubernetes](https://kubernetes.io/docs/getting-started-guides/minikube/).
I'll be doing local dev work so `minikube` makes sense, and I develop on OSX and will use Virtualbox for my Hypervisor.
Here we're installing `minikube` in which will run our Kubernetes cluster and `kubectl` which is a CLI for interacting
with the cluster:

```sh
$ brew install kubectl
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.19.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Then we'll start `minikube` and deploy a simple "hello world" pod. We will then expose the ports of that
pod so we can ping it:

```sh
$ minikube start
$ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
$ kubectl expose deployment hello-minikube --type=NodePort
```

We can then poll with `kubectl get pod` until the pod is ready:

```sh
$ kubectl get pod
NAME                             READY     STATUS              RESTARTS   AGE
hello-minikube-938614450-p0lqp   0/1       ContainerCreating   0          17s

$ kubectl get pod
NAME                             READY     STATUS    RESTARTS   AGE
hello-minikube-938614450-p0lqp   1/1       Running   0          27s
```

And finally ping the pod and then tear down minikube:

```sh
$ minikube service hello-minikube --url
http://192.168.99.100:30526

$ curl http://192.168.99.100:30526
CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.99.100:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=*/*
host=192.168.99.100:30526
user-agent=curl/7.51.0
BODY:
-no body in request-

$ minkube stop
```

Okay, let's boot `minikube` back up and toss a docker image up there. We'll write a
simple Ruby server with a signal handler:

```rb
# server.rb
trap "TERM" do
  $stderr.puts "received TERM, handling gracefully"
  sleep 5
  exit(0)
end

$stderr.puts "starting server, pid #{Process.pid}"

n = 0

loop do
  sleep 5
  n += 5
  $stderr.puts "alive for #{n} seconds"
end
```

And a simple Dockerfile to go with that:

```sh
# Dockerfile
FROM ruby:2.4-slim
COPY server.rb .
CMD ruby server.rb
```

We can run these just for a sanity check:

```sh
$ docker build .
Sending build context to Docker daemon 34.82 kB
Step 1/3 : FROM ruby:2.4-slim
2.4-slim: Pulling from library/ruby
10a267c67f42: Pull complete
0aaa89427703: Pull complete
4e4351445696: Pull complete
607f837da88c: Pull complete
37eaad0c7b0b: Pull complete
80bffd3c1c24: Pull complete
Digest: sha256:88c87f5e110db3be63cfcf27d2193ff2fa29497733d10e4a41747b3320e5dadb
Status: Downloaded newer image for ruby:2.4-slim
 ---> 8a007544be29
Step 2/3 : COPY server.rb .
 ---> 681d8e362105
Removing intermediate container b1145fb26c45
Step 3/3 : CMD ruby server.rb
 ---> Running in 39873c742f16
 ---> 945f0ff34a4f
Removing intermediate container 39873c742f16
Successfully built 945f0ff34a4f

$ docker run -it 945f0ff34a4f
starting server, pid 8
alive for 5 seconds
```

Cool. Let's push this up to `minikube`. `minikube` runs locally, so we just need to switch over our
Docker CLI to use minikube's docker daemon. Then we can build the image on that daemon. Finally we'll
create a pod deployment with that image:

```sh
$ eval $(minikube docker-env)
$ docker build -t signal-server .

$ docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
signal-server                                          latest              d44985234c2f        43 seconds ago      224 MB
ruby                                                   2.4-slim            8a007544be29        3 days ago          224 MB
gcr.io/google_containers/kubernetes-dashboard-amd64    v1.6.0              416701f962f2        2 months ago        109 MB
gcr.io/google-containers/kube-addon-manager            v6.4-beta.1         85809f318123        2 months ago        127 MB
gcr.io/google_containers/k8s-dns-sidecar-amd64         1.14.1              fc5e302d8309        2 months ago        44.5 MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64        1.14.1              f8363dbf447b        2 months ago        52.4 MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.1              1091847716ec        2 months ago        44.8 MB
gcr.io/google_containers/echoserver                    1.4                 a90209bb39e3        11 months ago       140 MB
gcr.io/google_containers/pause-amd64                   3.0                 99e59f495ffa        12 months ago       747 kB

# NOTE: I had to add the --image-pull-policy=IfNotPresent line to prevent Kubernetes
# from trying to pull from a Docker registry (which I didn't have up). YMMV with that
$ kubectl run signal-server --image=signal-server --image-pull-policy=IfNotPresent

$ kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
signal-server-4076204847-kr3pc   1/1       Running   0          3s
```

We can tail the logs:

```sh
$ kubectl logs -f signal-server-4076204847-kr3pc
starting server, pid 6
alive for 5 seconds
alive for 10 seconds
alive for 15 seconds
```

And then teardown the pod and we should see our signal handler called:

```sh
$ kubectl delete deployments --all
$ kubectl logs -f signal-server-4076204847-kr3pc
starting server, pid 6
alive for 5 seconds
alive for 10 seconds
alive for 15 seconds
alive for 20 seconds
alive for 25 seconds
alive for 30 seconds
alive for 35 seconds
```

Anddd it doesn't work. Hmm. I had heard in passing about Docker not getting signals when something was specified
as a `CMD`. I decided to try the `exec` form for that in my Dockerfile: `CMD ["ruby", "server.rb"]`

Now if we rebuild the image and re-run the test:

```sh
$ kubectl delete deployments --all
$ kubectl logs -f signal-server-4076204847-kr3pc
starting server, pid 1
alive for 5 seconds
alive for 10 seconds
received TERM, handling gracefully
```

Perfect! Notice also that our server is being started on pid 1. In the previous example we were on pid 6, so I'm
under the impression that Kubernetes essentially does a `kill -s TERM 1`. In the previous example, pid 1 was my `init`
process, so the signal never propagated to my server. In this case, Docker must be calling [exec(3)](https://linux.die.net/man/3/exec).
Instead of forking off a new subprocess, `exec` will **replace** the current process with the process defined by the
args, thus retaining the pid of 1 and making it the recipient of signals from Kubernetes.

So there you have it - two birds for one. We learned a little bit about how to work with Kubernetes and we also
found that we'll need to use `exec` form in our Dockerfile in order to properly receive signals. Now to tackle some
deeper Kubernetes topics like autoscaling! See you next time!
