---
layout: post
title: "Setting Up a Team Documentation Server Using YARD"
date: 2015-06-09 18:21:45 -0400
comments: true
categories: ruby
---

Our team has been using YARD syntax in to document our Ruby code for awhile now. It's a great way to document what methods do, which parameters they accept, and what the return. Here's an example of what a YARD doc looks like:

```rb
# Converts the object into textual markup given a specific format.
#
# @param format [Symbol] the format type, `:text` or `:html`
# @return [String] the object converted into the expected format.
def to_format(format = :html)
  # format the object
end
```

One of the best things about YARD is that it comes with a built-in web server that will compile and host all of your documentation. This makes it very easy to search and browse the code even if you're not very familiar with it.

```
yard server
```

By default YARD will host the server on port 8808. Depending on your host machine's configuration, you may just be able to go to your host IP on that port, but for us we had to use nginx to act as a proxy on port 80 to our YARD server.

```sh
# in /etc/nginx/sites-enabled/yard.conf
server {
  listen 80;
  server_name localhost;

  location / {
    proxy_pass http://127.0.0.1:8808;
  }

  client_max_body_size 4G;
  keepalive_timeout 10;
}
```

Restart the nginx service:

```sh
sudo service nginx restart
```

Now you should be able to simple enter your host IP in the browser and see your beautiful YARD app! One problem though: what happens when you push new code? Here's a quick shebang script for pulling updates and restarting the server. It assumes your host machine has been setup with rbenv and git, but it should be easy to modify for whatever your use case is.

```sh
# in restart_server

#! /usr/bin/env bash
echo 'Killing yard server...'
ps aux | grep 'yard server' | grep -v 'grep' | awk '{ print $2 }' | xargs kill

echo 'Removing old docs...'
# YARD caches the compiled docs in .yardoc/
rm -rf .yardoc

echo 'Pulling new code...'
cd /home/vagrant/yard && git pull

echo 'Starting yard server...'
# We want this to work with cron, so we do all the
# rbenv setup that we usually do when the shell starts
export PATH=/home/vagrant/.rbenv/shims:/home/vagrant/.rbenv/bin:/usr/bin:$PATH;
eval "$(rbenv init -)"

# Daemonize the server so it runs in the background
yard server -d
```

Give the script executable permissions:

```sh
chmod +x restart_server
```

Then give it whirl!

```sh
./restart_server
```

You can test it by trying to hit the server in the browser or by looking for the process with

```sh
ps aux | grep yard | grep -v grep
```

Restarting your server by hand might seem like a lot of fun, but having it restart automatically is even more fun! That's where cron comes in. The following cron job will run our restart script every half hour. It will also log any output to `restart_server.log` in case we have to debug any issues.

```sh
crontab -e

# in editor

# Restart server every half hour
# Redirect output to log file for debugging
0,30 * * * * ~/yard/restart_server > ~/yard/restart_server.log 2>&1
```

And that's all there is to it! Cron will automatically pick up your changes; no need to reload anything. Next we plan to work on a job that will parse our documentation and alert us of parse errors or missing documentation when we make changes in our pull requests. Stay tuned!
