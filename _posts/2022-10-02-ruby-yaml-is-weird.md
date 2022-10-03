---
title: (Ruby) YAML is weird
date: 2022-10-02 20:00:00 +0200
categories: ruby yaml
---

To my big suprise, turns out that `on` and `off` are supported boolean values in YAML:

```
$ ruby -ryaml -e "puts YAML.load('on: hello')"
{true=>"hello"}
```

So `YAML.load('on: hello')` is the same as `YAML.load('true: hello')` 🤔
YAML is usually used in configuration files because it is easier to read and navigate than JSON but I also feel like some of the parsing behavior is unexpected as this is the second time I'm taking time to write down some thoughts about how YAML parsing in Ruby surprised me (see [other post]({% post_url 2020-10-13-ruby-symbols-in-yaml %})).

So does the YAML parser in the Ruby stdlib, Psych, have the ability to parse YAML in a more limited JSON-esque manner?

The YAML spec mentions [JSON Schema](https://yaml.org/spec/1.2.2/#102-json-schema) which is different than JSON Schema used for validating JSON documents.
Schemas in the context of YAML parsing is described as

> A YAML schema is a combination of a set of tags and a mechanism for resolving non-specific tags.

As far as I can tell Psych does not support JSON Schema and I'm also not sure if that would even solve the challenges I have experienced with it. Ultimately I would like to have a simplified YAML parser which just

- understands JSON scalar values: string, number, boolean and null
- supports objects
- treats all object keys as strings
- supports arrays

Support for YAML aliases and anchors might also be useful but that's basically all I would expect from a YAML parser for most use cases, especially in the context of configuration files.

When time permits I will try to dedicate some time in the near future to look into what it would take to build this simplified YAML parser. Building any kind of parser always feels overwhelming for me but hopefully it will be possible to leverage Psych to achieve this. Wish me luck!
