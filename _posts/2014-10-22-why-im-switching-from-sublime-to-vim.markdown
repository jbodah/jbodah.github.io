---
layout: post
title: "Why I'm Switching from Sublime to Vim"
date: 2014-10-22
comments: true
categories: vim
---

I love Sublime. It's been my editor of choice for the past 4 years and is superior to most other modern editors. I consider myself a power user as I create my own snippets and my own build tools and am fluent with almost all of the keybinds. The only things I haven't done is make my own plugin or color scheme. My skills with Sublime are high enough where it pains me to watch others fumble around instead of doing a CMD+p CMD+r fuzzy search to get where they want.

But it's not enough. I feel like I've peaked out and there are still things I could improve. Some examples

- Vertical movement: Sublime is great at skipping words and moving to the ends of lines. It also has unparalleled cross file movement and movement to symbols. The only vertical movement I have though is page up/down, file beginning/end, and arrow keys. And that sucks.

- Arrow keys: I still find myself using the arrow keys in Sublime which always takes me away from the action. There is vi mode, but if I'm going to use that the  why not just use vim

- Cross platform. I can do basic stuff in vim pretty easily when I ssh, but I'm not fluent. Moreover, being able to choose a VM development workflow where I say into my dev environment is really appealing. This is not easy in Sublime

- Extensibility: This is the nail in the coffin. Sublime's extensibility feels tacked on and hacky. I can't get direct shell integration, and the configuration-based build system feels crappy. Plugins need to be written in Python which also kind of sucks working in a Ruby environment. Documentation for all of this is sketchy at best too. Maybe if it was open source, but it's not and - sadly - probably never will be

- Open source: I've come to really appreciate open source software. Don't like it? Change it.  Better yet, someone else probably already has. Not sure how something works? Spotty documentation? Go look at the source. This is Sublime's biggest downfall and will likely lead to its death in a few years

So I'll be switching to Vim here soon. I'm sure it will be a long, painful experience. I'll miss all my awesome keybinds from Sublime a lot, but it's the right decision in the long run. I hope to post Vim plugins and other things that make more Sublime-like
