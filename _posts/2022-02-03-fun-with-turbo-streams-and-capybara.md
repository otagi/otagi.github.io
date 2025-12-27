---
title: Fun with Turbo Streams and Capybara
tags: Code
---

Using and testing [Turbo Streams][ts] in a [Rails][rails] app is quite fun, but comes with its share of quirks, which are not always well documented.

Let’s explain some of these with a [very simple app][app], pompously titled “The Rodents Encyclopedia”. It consists of a dashboard displaying the number of rodents in the database, and a standard [CRUD][crud] for these rodents. When a rodent is added, the dashboard counter is automagically updated.

<img src="/assets/images/blog/rodents_encyclopedia.png" alt="The Rodents Encyclopedia app" width="100%"/>

## The system spec

The [feature][feature] is described in a system spec with [Capybara][capy] and [RSpec][rspec]:

```ruby
# spec/system/rodents_counter_spec.rb

feature 'Rodents Counter', puma: true, action_cable: :inline, active_job: :inline do
  background do
    driven_by :selenium, using: :headless_chrome
  end

  given!(:existing_rodent) { create(:rodent, name: 'Chipmunk') }

  scenario 'Creating a rodent updates the dashboard rodents counter' do
    visit '/'

    expect(page).to have_text 'There are 1 rodents in the encyclopedia!'

    new_window = open_new_window
    within_window new_window do
      visit '/rodents/new'
      fill_in 'Name', with: 'Squirrel'
      click_on 'Create Rodent'
    end

    expect(page).to have_text 'There are 2 rodents in the encyclopedia!'
  end
end
```

First peculiarity, the feature declaration:

```ruby
feature 'Rodents Counter', puma: true, action_cable: :inline, active_job: :inline do
```

`puma: true` is a tag defined in a [support file][capybara.rb] to use [Puma][puma] as the Capybara server. It should help running concurrent requests.

`action_cable: :inline` is a tag defined by RSpec. I’m not sure yet if that one is required over the standard `:async` or `:test` adapters. Feel free to experiment on your project.

`active_job: :inline` is a tag defined in [another support file][inline_active_job.rb]. Broadcast methods ending in `later` will start an `ActiveJob`. If the job is not run immediately during the spec, Capybara won’t see its result. So this tag is essential to not lose your sanity in debug sessions.

Next, we select the Selenium driver, because the default `rack_test` one won’t run Javascript code:

```ruby
driven_by :selenium, using: :headless_chrome
```

The actual spec code is now ready to run. It will first check that the dashboard counter is in its initial state, then proceed to create a rodent in a separate browser window, and finally check that the counter has been updated. Note that Capybara does not reload the dashboard page during the whole process, which is exactly the use case we want to test.

## The broadcast

Triggering the counter update is done in the [Rodent model][rodent.rb] whenever the record changes:

```ruby
# app/models/rodent.rb

class Rodent < ApplicationRecord
  after_commit :update_rodents_counter

  private

  def update_rodents_counter
    broadcast_replace_later_to :dashboard, target: 'rodents-counter', partial: 'dashboard/rodents_counter', locals: {rodent: nil}
  end
end
```

Let’s decompose the `broadcast_replace_later_to` call:

-  `replace` is the action relevant to the use case (there’s also `append`, `remove`, etc.). 
-  `later` wraps the call in a job so that the partial rendering will not slow down the web server response. 
-  `to` allows to pass a channel scope instead of the default model instance. 
-  `:dashboard` is our channel scope (see the view below). 
-  `target:` is the HTML id of the element to be replaced. 
-  `partial:` is the partial to render and send over the wire. 
-  `rodent: nil` overwrites the default partial locals set to the model instance. This is required because otherwise, when the record is destroyed, the generated broadcast job will abort with a deserialization error.

## The stream

The client subscribes to the channel in the [dashboard view][show.html.slim]:

```slim
/ app/views/dashboard/show.html.slim

= turbo_stream_from :dashboard

h1 Dashboard

== render 'rodents_counter'
```

The `:dashboard` part is the channel scope used earlier in the broadcast.

In the [rodents counter][_rodents_counter.html.slim] partial, the `rodents-counter` id is the target used in the broadcast:

```slim
/ app/views/dashboard/_rodents_counter.html.slim

p#rodents-counter There are #{Rodent.count} rodents in the encyclopedia!
```

That whole partial is rendered server-side and sent over the wire to the subscribed clients. Hotwire’s hidden Javascript code will take care of the replacement in the DOM.

That’s it, really. A few lines of code to update a page element. No need to write a dedicated Channel class or a Stimulus controller. Less code, and hopefully less bugs.

[capy]: https://teamcapybara.github.io/capybara/
[crud]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[puma]: https://puma.io
[rails]: https://rubyonrails.org
[rspec]: https://rspec.info
[ts]: https://turbo.hotwired.dev/handbook/streams

[app]: https://github.com/otagi/turbo-spec
[feature]: https://github.com/otagi/turbo-spec/blob/main/spec/system/rodents_counter_spec.rb
[inline_active_job.rb]: https://github.com/otagi/turbo-spec/blob/main/spec/support/inline_active_job.rb
[capybara.rb]: https://github.com/otagi/turbo-spec/blob/main/spec/support/capybara.rb
[rodent.rb]: https://github.com/otagi/turbo-spec/blob/main/app/models/rodent.rb
[show.html.slim]: https://github.com/otagi/turbo-spec/blob/main/app/views/dashboard/show.html.slim
[_rodents_counter.html.slim]: https://github.com/otagi/turbo-spec/blob/main/app/views/dashboard/_rodents_counter.html.slim
