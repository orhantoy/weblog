---
title: "Contributing to Rails: new numericality validator option"
date: 2021-12-21 12:00:00 +0100
categories: ruby rails opensource
---

Once in a while I make contributions to open source projects, and I'm extra excited to make contributions to Ruby on Rails. This time I decided to document the process to remove some of the mystery for people who are just starting their software development journey (even experienced developers could be intimidated by working in the open).

A few weeks ago I was reviewing a coworker's pull request in relation to the [Elastic crawler](https://www.elastic.co/web-crawler). We had to validate a `max_crawl_depth` integer, and I would normally do that with something like the following:

```ruby
class CrawlConfig
  include ActiveModel::Model

  attr_accessor :max_crawl_depth

  validates :max_crawl_depth, numericality: { only_integer: true, greater_than: 0 }

  # ...
end
```

Looks like it should work but it turns out that the numericality validator in Rails does not require `max_crawl_depth` to be an actual `Numeric` value - a string that looks like a number would also work. But if we ended up accepting a string value, we would have a potential issue further down in the execution of the program.

I saw this as an opportunity to propose a contribution to Rails. Where do you start with something like that?

- You can find the same information in multiple places but for Rails it probably makes sense to start in the [guides](https://guides.rubyonrails.org/contributing_to_ruby_on_rails.html#contributing-to-the-rails-code).
- For smaller open source projects you might be able to find that information in the README or CONTRIBUTING files.

The pull request for this specific change can be found [here](https://github.com/rails/rails/pull/43914).
To document the process in even greater detail, I also recorded my screen while working on this change and that video can be found [here](https://youtu.be/cmDF2bva1vY).
