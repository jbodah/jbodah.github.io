---
layout: post
title: "Browser Automation In Ruby"
date: 2014-11-12
comments: true
categories:
---

In my last job we used Watir for our automation scripts. If you haven't used it before, it's basically a wrapper for Selenium that allows you to script user interactions to a browser instance. It got the job done, and it was really useful for awhile (we wound up making some major architectural shifts and I never had the time to get it back up and running). The problem with Watir is that it's pretty much dead now. The documentation is sparse and the syntax is inconsistent at times.

The other big automation library for Ruby is Capybara. Capybara is a sexy DSL with a lot of magic going on. I like it quite a bit, but I haven't used it much outside of a Sinatra app. It's kind of heavyweight for what I want, but would be a good choice for a reasonably sized test suite.

There's a third option too: using Selenium directly. I'd never used Selenium directly until tonight, and I have to say I'm pretty impressed. The API is clear and concise, and aside from getting hung with some versioning between the gem and my browser, things went really smoothly. Best of all, it's lightweight and easy to make it feel Capybara-esque with some simple syntactic sugar. Below is what I came up with for signing up a test account for manual testing. Some parts are a little ugly, but for an hour of work tops including getting setup I'm pretty happy:

```ruby
require 'selenium-webdriver'

driver = Selenium::WebDriver.for :firefox

define_method :el do |hash|
  key = hash.keys.first
  driver.find_element(key, hash[key])
end

define_method :to do |*args|
  driver.navigate.to(*args)
end

# First logout of Google
to 'http://google.com'
el(id: 'gb_70').click

# Login to Google
el(id: 'Email').send_keys @google_user
el(id: 'Passwd').send_keys @google_password
el(id: 'signIn').click

# Login to Backupify
to @url
el(css: '.google-brand').click

# Wait for OAuth
sleep(1) until driver.current_url[/settings\/domain/]

# Signup
el(id: 'country').send_key 's' # Any country
el(name: 'commit').click

# Wait for page load
sleep(1) until driver.current_url[/settings\/domain\/accounts/]

# Add user
el(css: 'tr.row-odd:nth-child(1) &gt; td:nth-child(1) &gt; input:nth-child(1)').click
el(css: '#domain_users &gt; header:nth-child(1) &gt; div:nth-child(2) &gt; a:nth-child(1)').click
el(name: 'commit').click

# Go to Qless web
to @qless if @qless
```
