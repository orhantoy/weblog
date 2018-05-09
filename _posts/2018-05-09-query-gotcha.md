---
title: Query gotcha
date: 2018-05-09 11:00:00 +0200
categories: rails
---

Not so long ago I ran into an issue with a method like this:

```ruby
class SomeRecord < ActiveRecord::Base
  class << self
    def autocomplete_search(title: nil)
      if title.present?
        where("title ILIKE ?", "%#{title}%")
      else
        self
      end
    end
  end
end
```

At first glance it looks and works fine:

```ruby
SomeRecord.where(some_field: "value").autocomplete_search(title: "Hello") # => ✅
SomeRecord.autocomplete_search(title: "Hello").where(some_field: "value") # => ✅
```

But there is a problem:

```ruby
SomeRecord.where(some_field: "value").autocomplete_search(title: "")
  # => SELECT * FROM "some_records" ⚠️
SomeRecord.autocomplete_search(title: "").where(some_field: "value")
  # => SELECT * FROM "some_records" WHERE "some_records"."some_field" = ? ✅
```

Now the first query returns an unexpected result without failing. Why?
When `title.blank?` we return `self` in `SomeRecord.autocomplete_search` which is equivalent to `SomeRecord.all`.

The same happens in the second query but because we chain `where(some_field: "value")` afterwards we still get the expected result.

By replacing `self` with `all` in `SomeRecord.autocomplete_search`,

```ruby
class SomeRecord < ActiveRecord::Base
  class << self
    def autocomplete_search(title: nil)
      if title.present?
        where("title ILIKE ?", "%#{title}%")
      else
        all # <-- ✌️
      end
    end
  end
end
```

we get the expected result.
