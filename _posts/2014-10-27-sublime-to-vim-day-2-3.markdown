---
layout: post
title: "Sublime to Vim: Day 2 & 3"
date: 2014-10-27
comments: true
categories:
---

Day 1 was pretty easy, but the day 2 was more difficult. I found myself really limited by what vim offers out of the box. Some of the configuration notation is pretty archaic, and it's a struggle to get what you want if you don't have a really solid understanding of how to do it.

I'm not really fond of macvim. I lose some time context switching between the terminal windows and the editor which I feel kind of defeats the purpose of vim. Right now I'm relying on tabbing through my windows and it's working okay, but it would be the first thing I would change. My color schemes look awful when called in the shell. I tried using iTerm instead of the default OSX Terminal and that didn't help things either (although I do like iTerm a bit more even if it does need to be tweaked to support OSX keybindings).

My current work screen is macvim on one half of the screen, a Guard instance in a corner, and my general iTerm in the other corner. My second monitor is for HipChat and my browser. I wish I could have vim, Guard, and my general terminal all together in the same spot and have it function okay, but it's going to take a bit more work.

As far as editing goes, I've stumbled across some really important tools. CtrlP is a must and gives you goto-anything. Supertab gives autocomplete. Airline gives a really nice status bar. Custom keybinds are really what make vim for me though. I'm still learning and want to start writing my own Ruby scripts for keybinds.

Ag, or the Silver Searcher, is the closest I could get to find/replace across many files. It's still not as good as Sublime though. My real beef right now is that it's not easy enough to do a renaming refactor. Cross file editing in general is a bit too difficult right now and nowhere near as seamless as Sublime.

That's all I have for now. I'm going to stick with it even though macvim is kind of annoying and cross file management is difficult. I think these might get better in time. For reading, I highly (!) recommend [Learn Vimscript the Hard Way](http://learnvimscriptthehardway.stevelosh.com/). It covers a lot of really important stuff about how to use vim as well as the philosophy of vim (you should make things as efficient as possible). Do I think you need to switch to vim to get this? No way. In fact, I probably would've stuck with Sublime if it hadn't been for the shell integration and cross platform support of vim. I might even still use Sublime for heavy cross file work, who knows.
