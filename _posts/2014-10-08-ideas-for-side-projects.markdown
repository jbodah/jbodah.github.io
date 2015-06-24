---
layout: post
title: "Ideas for Side Projects"
date: 2014-10-04
comments: true
categories:
---

Some ideas I had for side projects (since I keep leaving my notes at work):

  * A stream-based watch task library (like Guard)
    * should be able to run recent files, run periodically, run specific file via CLI
    * should be able to compose plugins easily (e.g. I should be able to easily throw SimpleCov in front of my Minitest tasks)
  * Git hook framework
    * Something to make it easy to switch between Git projects and maintain my hooks
    * Also hooks for byebug/pry detectors
  * Ruby Spy framework
    * Spies work like mocks but are transparent to the application
    * This allows us to test recursion easier and doesn't have the problem of mocks where they become outdated when the mocked object changes; it simply tests the interface and records each call
