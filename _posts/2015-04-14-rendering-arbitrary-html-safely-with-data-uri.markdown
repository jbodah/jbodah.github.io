---
layout: post
title: "Rendering Arbitrary HTML Safely with data-uri"
date: 2015-04-14 10:58:22 -0400
comments: true
categories: html
---

I was recently working on a message content view that needed to render arbitary HTML from an e-mail as a sort of preview to what that e-mail contained.
Just rendering the HTML straight to the page can screw up your styles and HTML (for example, if an e-mail had HTML tags that weren't properly closed).
Browsers try to do all sorts of things to "fix" your bad HTML too which can cause headaches.

A better approach is to encapsulate the HTML body in an iframe.
Normally you need to specify an endpoint via the src attribute for the iframe, but we found a neat little trick of using a data-uri instead.

Here's a snippet from one of our angular templates:

{% raw %}
```js
<iframe ng-src="{{'data:text/html;charset=utf-8,' + encodeURIComponent(messageContent)}}"></iframe>
```
{% endraw %}

This means we don't need to create a separate endpoint and can send the HTML payload along in our normal JSON response.
The browser can just render the HTML directly in the iframe instead of making an extra request for it.

Now if you've done a lot of front-end dev then you might be thinking "does it work in IE?", and the answer is sadly "no, not for iframes".
IE currently supports link tags (e.g. for stylesheets), javascript, and images (see [http://caniuse.com/#feat=datauri](http://caniuse.com/#feat=datauri)).
So while IE is still a problem, this is a really neat solution that can save you a lot of pain if you can use it.

One last thing I'd like to note is that you'll probably want to protect your users by sanitizing the HTML on the server side.
Otherwise you might be creating a security hole by allowing people to do things like execute arbitrary Javascript on load.
Rails makes this pretty easy with `ActionView::Helpers::SanitizeHelper#sanitize`.
