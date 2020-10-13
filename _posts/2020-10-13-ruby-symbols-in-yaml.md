---
title: Ruby Symbols in YAML
date: 2020-10-13 23:00:00 +0200
categories: ruby yaml
---

JSON is nice and simple. Parsing

```json
{
  "ent_search": {
    "listen_host": "::0"
  }
}
```

with `JSON.parse(s)` would return `{"ent_search"=>{"listen_host"=>"::0"}}` as expected.

Parsing YAML is not as simple. What would parsing

```yaml
ent_search:
  listen_host: ::0
```

return?

## YAML.load

`YAML.load(s)` returns `{"ent_search"=>{"listen_host"=>:":0"}}`.

What is the difference? `YAML.load(s).dig("ent_search", "listen_host")` is a (Ruby) Symbol, in this case with the value `:":0"`.

## YAML.safe_load

`YAML.safe_load(s)` on the other hand, which is what you should use when parsing YAML from users, will raise an exception:

```
~/.rbenv/versions/2.5.8/lib/ruby/2.5.0/psych/class_loader.rb:97:in `find':
  Tried to load unspecified class: Symbol (Psych::DisallowedClass)
```

## Now what?

For now, I think the simplest option is to use `YAML.safe_load` and quote values starting with a semicolon:

```yaml
ent_search:
  listen_host: "::0"
```

But I would like there to exist a `YAML.json_load` method (with a better name) that would support the unquoted variant. Perhaps that's something to look into and propose for [ruby/psych](https://github.com/ruby/psych).
