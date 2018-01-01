---
title: "Fresh routes"
date: 2018-01-01 23:30:00 +0100
categories: rails
---

Let's say you have to __rebuild__ the UI of a part of your Rails application.
Often you'll want to keep the existing UI around so you can compare it with the new UI.

This can be accomplished easily by defining new routes.

Let's say that we have a controller for `posts` and the associated routes are defined as such.

{% highlight ruby %}
resources :posts, only: [:new, :create]
{% endhighlight %}

You want to rebuild the UI for the action `posts#new` - just create new, fresh routes!

{% highlight ruby %}
resources :posts, only: [:new, :create]
resources :v2_posts, controller: "posts_v2", only: [:new, :create]
{% endhighlight %}

There are several, nice side-effects with this approach:

- You can roll this out to production (semi-secretly) and also test things out there.
- The PR for this change will be mostly green.

When the change is ready to go public you deprecate the old routes, controllers and views.
And we should remember to tidy up by renaming the routes, controllers and views.
