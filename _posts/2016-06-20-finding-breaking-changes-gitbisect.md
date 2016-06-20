---
layout: post
title: 'Finding Breaking Changes with git-bisect'
date: 2016-06-20 10:34:11 -0400
comments: true
categories:
---

I recently came back to work from PTO to see that our build was broken. Having been away for a couple of days I was out of the
loop as to what changes had gone on in the time I was out.

First I made sure the failure was reproducable locally and consistent. I used [minitest-tagz](https://github.com/backupify/minitest-tagz)
to limit the scope of my test suite (shameless self-promotion).

```rb
tag :focus
should "not see admin message if already dismissed it" do
  Backupify::Redis::Frontend.dismiss_admin_message(@user)

  get :index, :user_id => @user.id

  assert_select ".header_stripe" do
    assert_select ".header_stripe_notice", :count => 0
  end
end
```

Once I found that the test was failing consistently, I decided to learn a little about [git-bisect](https://git-scm.com/docs/git-bisect).
In short, git-bisect is a tool that takes a range of commits and does a binary search on those commits. In my case, it will do a binary
search to find the earliest commit that breaks my tests.

As with most git commands, there are multiple formats that you can use, but here's the one I used:

```sh
git bisect start <LATEST_COMMIT> <EARLIEST_COMMIT>
git bisect run <SCRIPT>
# Copy bad commit
git bisect reset
```

It's important that `<EARLIEST_COMMIT>` be a commit that you think will be a good commit (e.g. passes tests).
Also note that `<SCRIPT>` is any "exit 0"-style script (most test runners).

Again, in my case this wound up being:

```sh
git bisect start HEAD HEAD~20
git bisect run spring rake test TEST=test/my_test.rb
# Copy bad commit
git bisect reset
```

This wound up giving me the breaking commit nice and quickly.
