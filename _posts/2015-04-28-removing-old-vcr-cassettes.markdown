---
layout: post
title: "Removing Old VCR Cassettes"
date: 2015-04-28 12:33:32 -0400
comments: true
categories:
---

Have a bunch of VCR cassettes and don't know what's still being used? This Minitest plugin is for you!

```rb
# we stick this in concerns/unused_cassettes.rb
module UnusedCassettes
  USED_CASSETTES = Set.new

  class << self
    def unused_cassettes
      Dir['test/fixtures/vcr/**/*.yml'].map { |d| File.expand_path(d) } - UnusedCassettes::USED_CASSETTES.to_a
    end
  end

  class BaseReporter
    class << self
      def report
        unused_cassettes = UnusedCassettes.unused_cassettes
        if unused_cassettes.any?
          vcr_root = "#{Rails.root}/test/fixtures/vcr/"
          puts "\nUnused cassettes:\n#{unused_cassettes.map {|f| f.sub(vcr_root, '')}.join("\n\t")}"
        else
          puts "\nHooray, all cassettes are being used!"
        end
      end
    end
  end

  module VCRExtensions
    def insert_cassette(name, options = {})
      USED_CASSETTES << VCR::Cassette.new(name, options).file
      super
    end
  end

  module MinitestExtensions
    def self.extended(base)
      original_load_plugins = base.method(:load_plugins)
      base.singleton_class.send(:define_method, :load_plugins) do
        original_load_plugins.call
        self.extensions << 'unused_cassettes'
      end
    end

    def plugin_unused_cassettes_init(options)
      self.reporter << UnusedCassettes::MinitestExtensions::Reporter.new
    end

    class Reporter < Minitest::AbstractReporter
      def report
        UnusedCassettes::BaseReporter.report
      end
    end
  end
end

# then in test_helper.rb
require 'concerns/unused_cassettes'

if ENV['SHOW_UNUSED_CASSETTES']
  VCR.extend(UnusedCassettes::VCRExtensions)
  Minitest.extend(UnusedCassettes::MinitestExtensions)
end
```
