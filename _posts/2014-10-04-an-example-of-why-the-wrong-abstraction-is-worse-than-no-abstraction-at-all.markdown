---
layout: post
title: "An Example of why the Wrong Abstraction is Worse than No Abstraction at all"
date: 2014-10-04
comments: true
categories:
---

Every now and then you'll hear someone mention that using the wrong abstraction is often more detrimental than no abstraction at all. I'd like to provide an example of why I think that is.

I'm currently going through the [Gilded Rose Refactoring Kata](https://github.com/jimweirich/gilded_rose_kata). In this exercise you're given a nasty conditional statement that you need to refactor into something more manageable. Thankfully Jim gives you tests already, so all you need to do is the refactoring parts.

```ruby
def update_quality(items)
  items.each do |item|
    if item.name != 'Aged Brie' && item.name != 'Backstage passes to a TAFKAL80ETC concert'
      if item.quality > 0
        if item.name != 'Sulfuras, Hand of Ragnaros'
          item.quality -= 1
        end
      end
    else
      if item.quality < 50
        item.quality += 1
        if item.name == 'Backstage passes to a TAFKAL80ETC concert'
          if item.sell_in < 11
            if item.quality < 50
              item.quality += 1
            end
          end
          if item.sell_in < 6
            if item.quality < 50
              item.quality += 1
            end
          end
        end
      end
    end
    if item.name != 'Sulfuras, Hand of Ragnaros'
      item.sell_in -= 1
    end
    if item.sell_in < 0
      if item.name != "Aged Brie"
        if item.name != 'Backstage passes to a TAFKAL80ETC concert'
          if item.quality > 0
            if item.name != 'Sulfuras, Hand of Ragnaros'
              item.quality -= 1
            end
          end
        else
          item.quality = item.quality - item.quality
        end
      else
        if item.quality < 50
          item.quality += 1
        end
      end
    end
  end
end

# DO NOT CHANGE THINGS BELOW -----------------------------------------

Item = Struct.new(:name, :sell_in, :quality)

# We use the setup in the spec rather than the following for testing.
#
# Items = [
#   Item.new("+5 Dexterity Vest", 10, 20),
#   Item.new("Aged Brie", 2, 0),
#   Item.new("Elixir of the Mongoose", 5, 7),
#   Item.new("Sulfuras, Hand of Ragnaros", 0, 80),
#   Item.new("Backstage passes to a TAFKAL80ETC concert", 15, 20),
#   Item.new("Conjured Mana Cake", 3, 6),
# ]
```

In my solution I wound up extracting the logic for each of the items into their own ItemUpdater class to take advantage of polymorphism. It's pretty clear looking at the code that we are switching based on a few cases, and polymorphism is the classical way to abstract these cases nicely. I find this to be much more readable (want to know what rules there are around Sulfuras? Go look at it's ItemUpdater class). Lastly, I also use a basic factory pattern to instantiate the correct ItemUpdater class.

```ruby
module ItemUpdaters
  class Base
    def initialize(item)
      @item = item
    end

    def update_quality; end

    private

    def increase_quality
      @item.quality += 1
      if @@max_quality
        @item.quality = [@@max_quality, @item.quality].min
      end
    end

    def decrease_quality
      @item.quality -= 1
      if @@min_quality
        @item.quality = [@@min_quality, @item.quality].max
      end
    end

    def self.max_quality(val)
      @@max_quality = val
    end

    def self.min_quality(val)
      @@min_quality = val
    end
  end

  class AgedBrie < Base
    max_quality 50

    def update_quality
      @item.sell_in -= 1
      increase_quality
      increase_quality if @item.sell_in < 0
    end
  end

  class NormalItem < Base
    min_quality 0

    def update_quality
      @item.sell_in -= 1
      decrease_quality
      decrease_quality if @item.sell_in < 0
    end
  end

  class BackstagePass < Base
    max_quality 50

    def update_quality
      @item.sell_in -= 1
      increase_quality
      increase_quality  if @item.sell_in < 5
      increase_quality  if @item.sell_in < 10
      @item.quality = 0 if @item.sell_in < 0
    end
  end
end

class ItemUpdaterFactory
  CLASS_MAP = {
    :default => ItemUpdaters::NormalItem,
    'Aged Brie'  => ItemUpdaters::AgedBrie,
    'Sulfuras, Hand of Ragnaros' => ItemUpdaters::Base,
    'Backstage passes to a TAFKAL80ETC concert' => ItemUpdaters::BackstagePass
  }

  def self.build(item)
    updater = CLASS_MAP[item.name] || CLASS_MAP[:default]
    updater.new(item)
  end
end

def update_quality(items)
  items.each {|i| ItemUpdaterFactory.build(i).update_quality}
end

# DO NOT CHANGE THINGS BELOW -----------------------------------------

Item = Struct.new(:name, :sell_in, :quality)

# We use the setup in the spec rather than the following for testing.
#
# Items = [
#   Item.new("+5 Dexterity Vest", 10, 20),
#   Item.new("Aged Brie", 2, 0),
#   Item.new("Elixir of the Mongoose", 5, 7),
#   Item.new("Sulfuras, Hand of Ragnaros", 0, 80),
#   Item.new("Backstage passes to a TAFKAL80ETC concert", 15, 20),
#   Item.new("Conjured Mana Cake", 3, 6),
# ]
```

But the refactoring is not the real thing I want to show you. What I really want to show you is the test file. You'll notice that the tests were all written to the original nasty implementation and have **ZERO** knowledge of the abstractions that I built. This is really important to understand - **the reason that this refactoring exercise was such a breeze is because the tests are decoupled from the implementation**.

```ruby
require 'rspec/given'
require 'gilded_rose'

describe "#update_quality" do

  context "with a single" do
    Given(:initial_sell_in) { 5 }
    Given(:initial_quality) { 10 }
    Given(:item) { Item.new(name, initial_sell_in, initial_quality) }

    When { update_quality([item]) }

    context "normal item" do
      Given(:name) { "NORMAL ITEM" }

      Invariant { item.sell_in.should == initial_sell_in-1 }

      context "before sell date" do
        Then { item.quality.should == initial_quality-1 }
      end

      context "on sell date" do
        Given(:initial_sell_in) { 0 }
        Then { item.quality.should == initial_quality-2 }
      end

      context "after sell date" do
        Given(:initial_sell_in) { -10 }
        Then { item.quality.should == initial_quality-2 }
      end

      context "of zero quality" do
        Given(:initial_quality) { 0 }
        Then { item.quality.should == 0 }
      end
    end

    context "Aged Brie" do
      Given(:name) { "Aged Brie" }

      Invariant { item.sell_in.should == initial_sell_in-1 }

      context "before sell date" do
        Then { item.quality.should == initial_quality+1 }

        context "with max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "on sell date" do
        Given(:initial_sell_in) { 0 }
        Then { item.quality.should == initial_quality+2 }

        context "near max quality" do
          Given(:initial_quality) { 49 }
          Then { item.quality.should == 50 }
        end

        context "with max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "after sell date" do
        Given(:initial_sell_in) { -10 }
        Then { item.quality.should == initial_quality+2 }

        context "with max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end
    end

    context "Sulfuras" do
      Given(:initial_quality) { 80 }
      Given(:name) { "Sulfuras, Hand of Ragnaros" }

      Invariant { item.sell_in.should == initial_sell_in }

      context "before sell date" do
        Then { item.quality.should == initial_quality }
      end

      context "on sell date" do
        Given(:initial_sell_in) { 0 }
        Then { item.quality.should == initial_quality }
      end

      context "after sell date" do
        Given(:initial_sell_in) { -10 }
        Then { item.quality.should == initial_quality }
      end
    end

    context "Backstage pass" do
      Given(:name) { "Backstage passes to a TAFKAL80ETC concert" }

      Invariant { item.sell_in.should == initial_sell_in-1 }

      context "long before sell date" do
        Given(:initial_sell_in) { 11 }
        Then { item.quality.should == initial_quality+1 }

        context "at max quality" do
          Given(:initial_quality) { 50 }
        end
      end

      context "medium close to sell date (upper bound)" do
        Given(:initial_sell_in) { 10 }
        Then { item.quality.should == initial_quality+2 }

        context "at max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "medium close to sell date (lower bound)" do
        Given(:initial_sell_in) { 6 }
        Then { item.quality.should == initial_quality+2 }

        context "at max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "very close to sell date (upper bound)" do
        Given(:initial_sell_in) { 5 }
        Then { item.quality.should == initial_quality+3 }

        context "at max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "very close to sell date (lower bound)" do
        Given(:initial_sell_in) { 1 }
        Then { item.quality.should == initial_quality+3 }

        context "at max quality" do
          Given(:initial_quality) { 50 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "on sell date" do
        Given(:initial_sell_in) { 0 }
        Then { item.quality.should == 0 }
      end

      context "after sell date" do
        Given(:initial_sell_in) { -10 }
        Then { item.quality.should == 0 }
      end
    end

    context "conjured item" do
      Given(:name) { "Conjured Mana Cake" }

      Invariant { item.sell_in.should == initial_sell_in-1 }

      context "before the sell date" do
        Given(:initial_sell_in) { 5 }
        Then { item.quality.should == initial_quality-1 }

        context "at zero quality" do
          Given(:initial_quality) { 0 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "on sell date" do
        Given(:initial_sell_in) { 0 }
        Then { item.quality.should == initial_quality-2 }

        context "at zero quality" do
          Given(:initial_quality) { 0 }
          Then { item.quality.should == initial_quality }
        end
      end

      context "after sell date" do
        Given(:initial_sell_in) { -10 }
        Then { item.quality.should == initial_quality-2 }

        context "at zero quality" do
          Given(:initial_quality) { 0 }
          Then { item.quality.should == initial_quality }
        end
      end
    end
  end

  context "with several objects" do
    Given(:items) {
      [
        Item.new("NORMAL ITEM", 5, 10),
        Item.new("Aged Brie", 3, 10),
      ]
    }

    When { update_quality(items) }

    Then { items[0].quality.should == 9 }
    Then { items[0].sell_in.should == 4 }

    Then { items[1].quality.should == 11 }
    Then { items[1].sell_in.should == 2 }
  end
end
```

Imagine now that you *started* with the implementation I created, and that your tests all referenced those abstractions (e.g. the Sulfuras tests directly called the ItemUpdaters::Sulfuras). Then refactoring these to some other pattern would be much more difficult because your tests would be breaking much more because they would be coupled with the implementation. Pulling up the methods on each of the classes would be a lot harder because your entire test suite would break due to directly referencing the ItemUpdater classes.

In essence, **we want to strive for tests that are decoupled from our implementation because it makes refactoring (i.e. changing that implementation later when we realize it's the wrong one) so much easier**. In the example above, the original Gilded Rose tests are extremely valuable because they only rely on the update_quality method. They let us abstract our refactoring implementation from the requirements spec meaning that our implementation can be easily changed. This is a critical part of why the "test-first" philosophy can work very well. Writing our tests after the implementation is done can make it harder to ensure that our tests are not coupled with our implementation.
