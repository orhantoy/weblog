---
title: "Exploring: composed_of"
date: 2018-07-19 21:00:00 +0200
categories: rails activerecord
---

I've never used `composed_of` or seen in it used in any of the Rails apps I've worked on.
The class method is [very well documented](http://api.rubyonrails.org/v5.2/classes/ActiveRecord/Aggregations/ClassMethods.html).
In this post I'll explore this, for me, unused method which on the surface looks useful.

So, what is the use case for `composed_of`? The documentation says

> Active Record implements aggregation through a macro-like class method called `composed_of` for representing attributes as value objects.

Aggregation in this case should not be confused with aggregate functions in SQL for doing things like `COUNT()`, `AVG()` and `SUM()`. A more clear definition could be that `composed_of` provides a way to represent a group of (one or more) DB columns as value objects.

Monetary values is a good use case for `composed_of`. Here's a runnable test case:

```ruby
# frozen_string_literal: true

begin
  require "bundler/inline"
rescue LoadError => e
  $stderr.puts "Bundler version 1.10 or later is required. Please update your Bundler"
  raise e
end

gemfile(true) do
  source "https://rubygems.org"

  git_source(:github) { |repo| "https://github.com/#{repo}.git" }

  # gem "rails", github: "rails/rails"
  gem "rails", "~> 5.2"
  gem "sqlite3"
  gem "money", "~> 6.12"
end

require "active_record"
require "minitest/autorun"
require "logger"

# This connection will do for database-independent bug reports.
ActiveRecord::Base.establish_connection(adapter: "sqlite3", database: ":memory:")
ActiveRecord::Base.logger = Logger.new(STDOUT)

ActiveRecord::Schema.define do
  create_table :products, force: true do |t|
    t.integer :price_in_cents
    t.string :currency
  end
end

class Product < ActiveRecord::Base
  composed_of :price, class_name: "Money", mapping: [%w(price_in_cents fractional), %w(currency currency_as_string)]
end

class ProductTest < Minitest::Test
  def test_product_price_is_set
    product = Product.create!(price: Money.new(100_00, "USD"))

    assert_equal "USD", product.currency
    assert_equal 100_00, product.price_in_cents
  end

  def test_product_price_can_be_queried
    Product.delete_all
    Product.create!(price: Money.new(100_00, "USD"))
    Product.create!(price: Money.new(10_00, "EUR"))
    Product.create!(price: Money.new(100_00, "DKK"))
    Product.create!(price: Money.new(10_00, "DKK"))

    assert_equal 1, Product.where(price_in_cents: 100_00, currency: "DKK").count
    assert_equal 1, Product.where(price: Money.new(100_00, "DKK")).count
  end
end
```

The Rails docs has a similar example with money. Other useful examples where value objects could be introduced is with weight, length and temperature, basically any case where we have a value + unit.

Depending on your problem domain `composed_of` can also be useful for representing 2D or 3D coordinates as `Point`s. Latitude and longitude coordinates are also a good candidate for use with `composed_of`.

## Final thoughts

I'll be more on the lookout for value objects in my code base as `composed_of` provides an easy way to expose them via DB columns.
