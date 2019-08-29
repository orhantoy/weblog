---
title: URL slug conflict validation
date: 2019-08-29 22:00:00 +0200
categories: rails urls
---

The username you choose on GitHub gives you a personal profile page under `github.com/<username>`. But a username like `about` would not be valid because `github.com/about` is the About page and therefore reserved.

There is a similar problem when user-defined slugs/names are used in URLs. When you create an [App Search](https://www.elastic.co/products/app-search) engine the name is used as part of a URL like `/as/engines/<engine-name>/documents`. `new` as an engine name wouldn't work because `/as/engines/new` is the URL for the page where you create a new engine.

In this blog post I'm going to explore solutions for this problem.

## The manual solution

A simple way to guard against conflicting URLs is to validate with a list of disallowed slugs/names.

## The automatic solution

`config/routes.rb` already defines the possible routes so here's how an automatic solution could look like:

```ruby
# frozen_string_literal: true

require "bundler/inline"

gemfile(true) do
  source "https://rubygems.org"

  gem "rails", "6.0.0"
  gem "sqlite3"
  gem "byebug"
end

require "action_controller/railtie"
require "active_record"
require "minitest/autorun"

ActiveRecord::Base.establish_connection(adapter: "sqlite3", database: ":memory:")
ActiveRecord::Base.logger = Logger.new(STDOUT)
ActiveRecord::Schema.define do
  create_table :accounts do |t|
  end

  create_table :engines do |t|
    t.belongs_to :account, null: false
    t.string :name, null: false

    t.index [:account_id, :name], unique: true
  end
end

class TestRailsApp < Rails::Application
  config.root = __dir__
  config.hosts << "example.org"
  config.session_store :cookie_store, key: "cookie_store_key"
  secrets.secret_key_base = "secret_key_base"

  config.logger = Logger.new($stdout)
  Rails.logger  = config.logger

  routes.draw do
    root to: "home#index"

    namespace :app_search, path: "as" do
      resources :engines, param: :name
    end
  end
end

module AppSearch
  class EnginesController < ActionController::Base
    # Define actions here.
  end
end

class Account < ActiveRecord::Base
  has_many :engines
end

class Engine < ActiveRecord::Base
  belongs_to :account

  validates :name, format: { with: /\A[a-z][a-z0-9_\-]*\z/ }
  validate :validate_name_results_in_recognized_path

  private

  def validate_name_results_in_recognized_path
    return if name_results_in_recognized_path?

    errors.add(:name, :inclusion, value: name)
  end

  def name_results_in_recognized_path?
    path = Rails.application.routes.recognize_path("/as/engines/#{name}")
    path[:controller] == "app_search/engines" && path[:action] == "show"
  rescue ActionController::RoutingError
    false
  end
end

class EngineTest < Minitest::Test
  def test_success
    account = Account.create!
    engine = account.engines.create!(name: "product-listing")

    assert_equal "product-listing", engine.name
  end

  def test_uniqueness
    account = Account.create!
    account.engines.create!(name: "product-listing")

    assert_raises('ActiveRecord::RecordNotUnique') do
      account.engines.create!(name: "product-listing")
    end
  end

  def test_url_conflict
    account = Account.create!
    engine = account.engines.new(name: "new")

    assert engine.invalid?
  end

  def test_name_with_slashes
    account = Account.create!
    engine = account.engines.new(name: "no/slash")

    assert engine.invalid?
  end
end
```

The downside is that `Rails.application.routes.recognize_path` is not a publicly documented method meaning it can't fully be relied on. But with a few tests, as shown in bottom of the snippet, you should be able to notice when that particular API breaks.
