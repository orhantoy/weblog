---
title: "Smart routes"
date: 2018-01-09 20:30:00 +0100
categories: rails
---

Constraints on routes can be a way to deal with resources that have different representations.

Imagine you have an `Asset` model for representing all types of files. You could subclass this model for a more specific purpose: `Logo < Asset`.

Nothing is stopping you from presenting both from `/assets/:id` but now you'll typically handle the different representations of `Logo` and `Asset` in a single controller. But with route constraints you can have the same URL route to different controllers depending on request parameters/properties:

```ruby
# config/routes.rb
require "route_constraints"

resources :assets, only: [:show], controller: "logos", constraints: RouteConstraints.logo_assets
resources :assets, only: [:show], constraints: RouteConstraints.regular_assets

# lib/route_constraints.rb
module RouteConstraints
  def self.logo_assets(param_key = :id)
    LogoConstraint.new(param_key)
  end

  def self.regular_assets(param_key = :id)
    -> (_request) { true }
  end

  class LogoConstraint
    attr_accessor :param_key

    def initialize(param_key)
      self.param_key = param_key
    end

    def matches?(request)
      begin
        Logo.find(request.parameters[param_key])
      rescue ActiveRecord::RecordNotFound
        false
      end
    end
  end
end
```

This technique can be used in combination with the technique described in [my previous post]({% post_url 2018-01-01-fresh-routes %}):

```ruby
# config/routes.rb
require "route_constraint"

resources :posts, controller: "posts_v2", only: [:new, :create], constraints: RouteConstraints.feature_flag("posts_v2")
resources :posts, only: [:new, :create]

# lib/route_constraints.rb
module RouteConstraints
  def self.feature_flag(key)
    FeatureFlagConstraint.new(key)
  end

  class FeatureFlagConstraint
    attr_accessor :key

    def initialize(key)
      self.key = key
    end

    def matches?(request)
      FeatureFlag
        .where(key: key)
        .where(resource: user_from_request(request))
        .exists?
    end

    private

    def user_from_request(request)
      # TODO: Extract the user from the request in some way.
      request.env["warden"].user
    end
  end
end
```
