---
title: Flaky Specs
date: 2021-09-29 13:00:00 +0200
categories: ruby specs testing
---

Once an application starts to get complex, the test suite will also start to grow, and you will most likely experience more and more flaky specs. In this post I'll explore an example of this from the test suite of Elastic Enterprise Search.

## Flaky?

If a spec/test fails in certain situations, at seemingly random times, it could be considered flaky. Ideally, you should try to fix this the first time you see it but sometimes it doesn't happen enough to warrant spending a lot of time on it. It's also not always easy to reproduce the issue on your local machine. The flaky spec blocking your PR could also be happening in a part of the application that you're not responsible for.

There are a lot of reasons not to worry about flaky specs until it starts to become an issue that affects a lot of people. Here I'll highlight one recent example and how that specific failure could've been avoided by thinking a bit broadly about the test suite and also by perhaps writing the non-test code a bit differently.

## Scenario

Here's a simplified version of the code and test for which we were experiencing specs that started to fail more frequently on CI but running the spec in isolation was working fine:

```ruby
require 'bundler/inline'

gemfile do
  source 'https://rubygems.org'
  gem 'rspec'
end

require 'yaml'

module EnterpriseSearch
  def self.product_version
    @product_version ||= Gem::Version.new(File.read('./product_version').chomp)
  end

  class ConfigParser
    attr_reader :parsed_config

    def initialize(file_name)
      @file_name = file_name
      @parsed_config = parse_config_file!
    end

    private

    attr_reader :file_name

    def parse_config_file!
      config = YAML.safe_load(File.read(file_name))

      if EnterpriseSearch.product_version >= Gem::Version.new('8.0')
        if config.key?('monitoring')
          raise "monitoring config is removed from 8.0 (your version: #{EnterpriseSearch.product_version})"
        end
      end

      config
    end
  end
end

require 'rspec/autorun'

RSpec.configure do |rspec|
  rspec.register_ordering(:global) do |items|
    if ENV['SPEC_ORDER'] == 'reverse'
      items.reverse
    else
      items
    end
  end
end

RSpec.describe EnterpriseSearch do
  describe '.product_version' do
    it 'should return version' do
      expect(EnterpriseSearch.product_version).to be_a(Gem::Version)
    end
  end
end

RSpec.describe EnterpriseSearch::ConfigParser do
  describe '#parsed_config' do
    it 'should parse' do
      expect(File).to receive(:read).and_return('hello: world')
      config_parser = described_class.new('banana.yml')
      expect(config_parser.parsed_config).to eq({ 'hello' => 'world' })
    end
  end
end
```

Running this file normally, the specs will not fail. But if you run it with a special environment variable, you will see it fail:

```bash
$ SPEC_ORDER=reverse ruby flaky_spec.rb
F.

Failures:

  1) EnterpriseSearch::ConfigParser#parsed_config should parse
     Failure/Error: @product_version ||= Gem::Version.new(File.read('./product_version').chomp)

     ArgumentError:
       Malformed version number string hello: world
     # flaky_spec.rb:12:in `product_version'
     # flaky_spec.rb:30:in `parse_config_file!'
     # flaky_spec.rb:20:in `initialize'
     # flaky_spec.rb:65:in `new'
     # flaky_spec.rb:65:in `block (3 levels) in <main>'

Finished in 0.01049 seconds (files took 0.09006 seconds to load)
2 examples, 1 failure

Failed examples:

rspec flaky_spec.rb:63 # EnterpriseSearch::ConfigParser#parsed_config should parse
```

The difference is in the **order** of the specs. Order-dependent specs are not something you want and luckily in this case the solution is mostly straightforward.

## Solution

Stubbing `File.read` with

```ruby
expect(File).to receive(:read).and_return('hello: world')
```

means it will also be stubbed for the `File.read` in `EnterpriseSearch.product_version`. As an added complexity, the product version is memomized/cached which means that in some cases it will try to read the product version from a file and sometimes it will use the memomized value.

If we change the above stub to

```ruby
expect(File).to receive(:read).with('banana.yml').and_return('hello: world')
```

we will still get an error:

```
     Failure/Error: @product_version ||= Gem::Version.new(File.read('./product_version').chomp)

       #<File (class)> received :read with unexpected arguments
         expected: ("banana.yml")
              got: ("./product_version")
```

This is not a big deal as we can do

```ruby
expect(File).to receive(:read).with('banana.yml').and_return('hello: world')
expect(File).to receive(:read).and_call_original
```

so other `File.read`s are called as normal.

## Retrospective

In retrospect, it seems much better to actually read a file from disk in the spec. With Ruby's `TempFile` we can create temporary files that should be cleaned up automatically. That way we can avoid the stub altogether.

Alternatively the `ConfigParser` class could also accept any IO object and in tests we could pass in `StringIO` so we avoid writing to and reading from disk.

Memoization, like we saw in `EnterpriseSearch.product_version`, also introduces pitfalls. In some scenarios you could do `EnterpriseSearch.instance_variable_set(:@product_version, nil)` to clear the cached value and thereby expose that there is actually a `File.read` hidden underneath. But having to think about that kind of stuff in unrelated specs is not easy to deal with so it would be preferable to having stubs/mocks that are local to the specs or let the related specs deal with resetting data so unrelated specs are unaffected.
