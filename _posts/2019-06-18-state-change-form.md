---
title: "Example: state change form"
date: 2019-06-18 22:00:00 +0200
categories: oop ruby
---

Here I'll explore some existing code where a change gets introduced and ultimately how a deeper understanding leads to a new realization and a better implementation.

The main concept in this example is a _Shipment_. A _Shipment_ is identified by a tracking number and also has a state, like whether it is in transit. This is the type of data you would see when you search for a parcel on the websites of UPS or FedEx.

The example concerns the following piece of code:

```ruby
module ShipmentViewHelper
  STATE_OPTIONS = [
    ["Created", Shipment::States::CREATED],
    ["Booked", Shipment::States::BOOKED],
    ["In transit", Shipment::States::IN_TRANSIT],
    ["Delivered", Shipment::States::DELIVERED],
    ["Cancelled", Shipment::States::CANCELLED],
  ]
end

class StateChangeForm
  include ActiveModel::Model

  attr_accessor :state, :tracking_number, :comment

  def available_state_options
    all_options = ShipmentViewHelper::STATE_OPTIONS.dup
    all_options.reject! { |(_, option_for_state)| option_for_state == Shipment::States::CREATED } unless state == Shipment::States::CREATED
    all_options
  end
end
```

`StateChangeForm` is a basic form model that holds various pieces of data related to a Shipment which is used to update the state of the shipment.
The interesting part is the method `StateChangeForm#available_state_options`. That method is used to list the available state values that the shipment can change to. The `CREATED` state is treated differently because it should not be able to go back to that state once it leaves it.

# Change

A new state is introduced: `CONFIRMED`. It is similar to the `CREATED` state in that it should not be possible to go back to the `CONFIRMED` state once it has left that state. A quick way to accomplish this:

```ruby
module ShipmentViewHelper
  STATE_OPTIONS = [
    ["Created", Shipment::States::CREATED],
    ["Confirmed", Shipment::States::CONFIRMED],
    ["Booked", Shipment::States::BOOKED],
    ["In transit", Shipment::States::IN_TRANSIT],
    ["Delivered", Shipment::States::DELIVERED],
    ["Cancelled", Shipment::States::CANCELLED],
  ]
end

class StateChangeForm
  include ActiveModel::Model

  attr_accessor :state, :tracking_number, :comment

  def available_state_options
    all_options = ShipmentViewHelper::STATE_OPTIONS.dup
    all_options.reject! { |(_, option_for_state)| option_for_state == Shipment::States::CREATED } unless state == Shipment::States::CREATED
    all_options.reject! { |(_, option_for_state)| option_for_state == Shipment::States::CONFIRMED } unless [Shipment::States::CREATED, Shipment::States::CONFIRMED].include?(state)
    all_options
  end
end
```

# Understand and refactor

Let's go a bit deeper and try to understand what the underlying idea is for `StateChangeForm#available_state_options`:

- if in state `CREATED`, the shipment can change to any state
- if in state `CONFIRMED`, the shipment can change to any state except for `CREATED`
- for the other states, the shipment can change to any state except for `CREATED` and `CONFIRMED`

This hints at the fact that depending on the state, we want to filter out (aka reject) some states.
This simplifies the implementation a lot:

```ruby
module ShipmentViewHelper
  STATE_OPTIONS = [
    # ... same as the previous step
  ]
end

class StateChangeForm
  include ActiveModel::Model

  attr_accessor :state, :tracking_number, :comment

  EXCLUDED_STATES_MAPPING = {
    Shipment::States::CREATED => [],
    Shipment::States::CONFIRMED => [Shipment::States::CREATED],
  }
  EXCLUDED_STATES_MAPPING.default = [Shipment::States::CREATED, Shipment::States::CONFIRMED]

  def available_state_options
    ShipmentViewHelper::STATE_OPTIONS
      .reject { |(_, option_for_state)| excluded_states.include?(option_for_state) }
  end

  def excluded_states
    EXCLUDED_STATES_MAPPING.fetch(state)
  end
end
```

*This could be further optimized to basically a hash lookup but that's beside the point.*
