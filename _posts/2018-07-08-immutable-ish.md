---
title: Immutable-ish
date: 2018-07-08 13:00:00 +0200
categories: rails rdbms
---

When modelling certain parts of an application backed by a relational database you are better of thinking of the data as immutable-ish; here follows 2 examples of what I mean by this.

## Example 1: User access

We have the following models:

```ruby
class Account < ApplicationRecord
end

class User < ApplicationRecord
end

class AccountMembership < ApplicationRecord
  belongs_to :account
  belongs_to :user

  scope :active, -> { where(revoked_at: nil) }
end
```

To give a user access to an account you would do

```ruby
AccountMembership.create!(account: account, user: user)
```

To revoke access for a user to an account you would do

```ruby
AccountMembership
  .active
  .where(account: account, user: user)
  .update_all(revoked_at: Time.zone.now)
```

When revoking access you are actually mutating the active `AccountMembership` records, hence why I call it immutable-ish 😎

The advantage of this data model is that the records themselves will contain timestamps (`created_at`: when was access given, `revoked_at`: when was access revoked) of what happened and when. This makes it easier to debug the timeline of events without resorting to also keeping track of actual events.

And lastly, we avoid dealing with duplicate records because we would check account access like this

```ruby
AccountMembership
  .active
  .where(account: account, user: user)
  .exists?
```

## Example 2: Plan subscription

For SaaS products you will usually need to associate accounts with pricing plans. This could be modelled like this:

```ruby
class Account < ApplicationRecord
  belongs_to :current_subscription, optional: true, class_name: "PlanSubscription"
  has_many :subscriptions, class_name: "PlanSubscription"
end

class PlanSubscription < ApplicationRecord
  belongs_to :account
end
```

When the account is initially subscribed to a plan we'll do

```ruby
ApplicationRecord.transaction do
  new_subscription = account.subscriptions.create!(plan_identifier: "small")
  account.update!(current_subscription: new_subscription)
end
```

And if the account upgrades or downgrades we'll do the exact same thing:

```ruby
ApplicationRecord.transaction do
  new_subscription = account.subscriptions.create!(plan_identifier: "large")
  account.update!(current_subscription: new_subscription)
end
```

Cancelling the subscription will be equally simple:

```ruby
account.update!(current_subscription: nil)
```

With this data model we'll know exactly when an account subscribed to a certain plan. This is especially important if we also want to keep track of invoices:

```ruby
class Invoice < ApplicationRecord
  belongs_to :account
  belongs_to :plan_subscription
end
```

This way every `Invoice` record will be associated with the `PlanSubscription` which was the `current_subscription` the account had at the time the invoice was generated.

With this example we are not mutating existing `PlanSubscription` records but we're updating `Account#current_subscription`.

## Summary

A database like [Datamic](https://www.datomic.com) uses the idea of immutability in its core architecture. Relational databases can mimick this behavior but I would recommend not using this approach for entities such as `User` because you lose the ability to reference data if you keep on creating new records.

A use case I did not mention here is that not mutating data will also allow us to easily revert back for instance if a user accidentally deletes something. Basecamp 3 does this with a concept they call _recordings_; you can see DHH mention it in [this video](https://www.youtube.com/watch?v=AoxoPfilKqE).
