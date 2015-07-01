---
layout: post
title: "Getting Started with Guard"
date: 2014-10-16
comments: true
categories: workflow, ruby, guard
---

Guard is a watch task gem that can run tasks when you change files (e.g. saving). It can run tests, static analyzers, and anything else you can think of.

1. Install gems

```
gem install spring rubocop flog guard guard-minitest guard-rubocop guard-flog
```


2. Add Guardfile

```ruby
guard :minitest, spring: true, all_on_start: false do
  # Application code
  watch(%r{^app/models/(.+)\.rb$}) { |m| "test/unit/#{m[1]}_test.rb" }
  watch(%r{^lib/(.+)\.rb$}) { |m| "test/unit/#{m[1]}_test.rb" }
  watch(%r{^app/controllers/(.+)\.rb$}) { |m| "test/functionals/#{m[1]}_test.rb" }

  # Test files
  watch(%r{^test/(unit|functionals|integration|script)/(.+)\.rb$}) { |m| "test/#{m[1]}/#{m[2]}.rb" }
end

guard :rubocop, all_on_start: false do
  # Application code
  watch(%r{^app/(.+)\.rb$}) { |m| "app/#{m[1]}.rb" }
  watch(%r{^lib/(.+)\.rb$}) { |m| "lib/#{m[1]}.rb" }
end

guard :flog do
  # Application code
  watch(%r{^app/(.+)\.rb$}) { |m| "app/#{m[1]}.rb" }
  watch(%r{^lib/(.+)\.rb$}) { |m| "lib/#{m[1]}.rb" }
end
```

3. Add .rubocop.yml

```
# Inherits everything else from defaul Rubocop config

Metrics/LineLength:
  Max: 100

Style/SpaceBeforeBlockBraces:
  Enabled: false

Style/SpaceInsideBlockBraces:
  Enabled: false

Style/SpaceInsideHashLiteralBraces:
  Enabled: false

Style/RaiseArgs:
  Enabled: false
```

4. Setup your binstubs with Spring

```
bundle exec spring binstub --all
```

(note: in order to get guard-minitest to run with spring then you must either give guard-minitest the `bundler: false` option or add spring to the Gemfile)

5. Add the Guardfile and .rubocop.yml to your global .gitignore

```
git config --global core.excludesfile '~/.gitignore'
vim ~/.gitignore
```

6. Start up Guard

```
guard
```

You can either save a file to watch Guard run or force Guard it to run via

```
c relative/path/to/file.rb
```
