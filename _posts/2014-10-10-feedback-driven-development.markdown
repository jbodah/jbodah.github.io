---
layout: post
title: "Feedback Driven Development"
date: 2014-10-10
comments: true
categories: workflow
---

I think it's safe to say that TDD is a mature topic in the programming community. It's a fairly heated topic that I feel divides people: you're either for TDD or you're against it. Those for it believe that TDD leads to more testable code which makes the code more modular and easier to use. Those against it claim that it tends to lead to monolithic test suites that lose purpose by being too tangled to an implementation which just makes refactoring harder. They also claim that it gives a false security that the system is really working (full unit test suite passes, but things still break due to not integration tests).

Personally I find it difficult to sit on either side of the fence. I like using TDD for feature work and checking requirements because I can write my spec ahead of time to ensure my system does everything I promised it would, but I often feel like I'm just stubbing and mocking too much when I don't have clear requirements (such as upgrading an API and having to detangle legacy code). I enjoy the immediate feedback I get from TDD, but it just kind of sucks when you don't know all of the requirements for what you need to be building. Moreover, writing good tests is really hard. Good tests are ones that don't break with your implementation, but I often find its impossible to get complete unit test coverage without doing some mocking and stubbing or without glossing over tests for one method. Are those tests still useful? I tend to think so, but it's true that they're fragile. Integration tests are much more durable, but they come at the cost of being slower and not being as isolated.

In addition, TDD can sometimes do more harm than good. I've seen code bases that are crying for tests, and TDD is a great way to introduce them there. I've also seen code bases with significant test debt that have way more test code than application code. The worst part is that the application code wasn't even  refactored well - it was just refactored to remove duplication rather than refactored to have a clear structure. Dependencies still polluted the system and TDD never led to strong abstractions.

So I'm bashing TDD here, but I don't mean to. It's a great tool when used properly (writing good tests is beyond the scope of this post). What I want to do is offer up a different solution: feedback driven  development.

Feedback driven development is a subset of TDD in a sense. The goal of feedback driven development is to use tools that give you constant feedback about the cde you're writing. This may be unit tests, coverage reports, code and style analysis - whatever you can think of. The point is that something is always giving you input to your code's quality.

For me, Guard is that tool. Guard will watch my file system for changes and run tasks on files when they are saved. My Guard setup has plugins for Minitest (for unit testing), Rubocop (for style analysis), and Flog (for code quality metrics). I also use SimpleCov for coverage in my test helper (not ideal, but it works for now). The point is that every time I save code I my get feedback on my test results and coverage, whether I'm abiding to our style guide, and on how complex my design is. This constant feedback loop helps me know I'm moving in the right direction and gives me all of things I like about TDD without the parts I don't.

If you haven't tried using a watch task tool like Guard then I definitely recommend that you try it out. It fits well in a TDD workflow and outside of one. The point I'm trying to make though is that you should use some sort of feedback cycle regardless of your preferred development approach. I'm almost certain you will enjoy it and write better code as a result
