---
title: Split tests
date: 2018-04-09 11:00:00 +0200
categories: rails testing
---

You can have more than one unit-test class _per_ class you want to test. I somehow had the mental limitation that one unit-test should test one class _entirely_ but this can lead to enormous test classes especially when working on legacy applications with few tests to begin with.

If you want to easily maintain and grow your test suite for legacy applications you should try splitting your tests.

Let's say you have the class `Account` with a lot of behavior - whether this is a bad or good thing does not matter much if that's the status quo. Example of how you could structure tests for that class:

```ruby
# test/models/account/billing_test.rb
class AccountBillingTest < ActiveSupport::TestCase
  # ...
end

# test/models/account/users_test.rb
class AccountUsersTest < ActiveSupport::TestCase
  # ...
end

# test/models/account/plan_test.rb
class AccountPlanTest < ActiveSupport::TestCase
  # ...
end

# ...
```

Eventually you would want to end up with classes with fewer responsibilities making this technique obsolete. But to get there having maintainable tests will help you.
