---
title: Models on Rails
date: 2018-11-08 22:00:00 +0200
categories: rails
---

By default Rails provides directories for the different kind of files you might need to build your application.
Developers usually agree on the purpose of `controllers`, `views`, `jobs` and `helpers` but what is `models` for?.

The Rails README has a good definition of what the model layer consists of in Rails:

> The _**Model layer**_ represents the domain model (such as Account, Product,
> Person, Post, etc.) and encapsulates the business logic specific to
> your application. In Rails, database-backed model classes are derived from
> `ActiveRecord::Base`. Active Record allows you to present the data from
> database rows as objects and embellish these data objects with business logic
> methods.
>
> Although most Rails models are backed by a database, models can also be ordinary
> Ruby classes, or Ruby classes that implement a set of interfaces as provided by
> the Active Model module.

When I started out with Rails I perceived the `models` directory as a home exclusively for Active Record backed classes. That's all fine and dandy... until your application needs to interact with an API. Where should the `DropboxUpload` class live?

My answer to that is: put it in `models`!

## Why models?

As the Rails README clearly states `models` is not only for AR-backed classes. Your domain model usually consists of things that talk with your database but this is not much different from talking with external services like Redis and 3rd APIs. So considering those classes as models make sense.

## More structure

In some cases though I wouldn't mind grouping related classes into a new directory. If for instance your application integrates with different kind of services in a *similar way* then it could make sense to group them. This can be done inside the `models` directory, usually by introducing a namespace (e.g. directory: `file_services`, module: `FileServices`). Alternatively if it is a core concept of your application it could also live directly under `app/`.

## Going deeper

You might be thinking that my suggestion is basically to put more stuff into `models`. But the more I think about it, it actually goes a bit deeper. A relatively big group of Rails developers introduce additional concepts like

- form objects,
- validator objects,
- service objects,
- query objects,
- presenters,
- etc

which in itself is not problematic. Introducing smaller, focused classes is usually a good thing. But I think you can still end with easily understandable and testable code by thinking in terms of models/resources.

## Conclusion

This discussion, like almost any discussion related to code, will be easier to understand when comparing actual code examples.
I have a lot of thoughts on this topic - instead of a very long post I'll try to dive into different subtopics in future posts.
