# [Turbolinks](https://github.com/turbolinks/turbolinks)

## Intro
Ever noticed those little loading bars at the top of some websites?

Youtube:
![Youtube Preview](/ruby/rails/turbolinks/assets/youtube_demo.gif)
Medium:
![Medium Preview](/ruby/rails/turbolinks/assets/medium_demo.gif)
Github:
![Github Preview](/ruby/rails/turbolinks/assets/github_demo.gif)

While they may not be using the Turbolinks library, they're leveraging the same principle of changing the page content without fully refreshing the DOM.

## Overview
_NOTE: This is basically just an abbreviated version of [the turbolinks readme](https://github.com/turbolinks/turbolinks).  If this is interesting to you, you're better off reading that._

Turbolinks is a way of giving a website all of the benefits of a single-page application without the overhead of a client-side Javascript framework.  The library allows the backend to remain unchanged while relying on javascript to intercept click events and render the page without incurring a full page load.  It'll be default in Rails 5 so we'd better get used to it now. :dancer:

Honestly just go to the Turbolinks repo if you want to read things, I don't word good.

## But How?
Like this:
1. Add gem to Gemfile
```ruby
gem 'turbolinks', '~> 5.0.0'
```
2. Do `bundle install`
3. Update application.js
```javascript
//= require turbolinks
```

That's it:
![Bonusly Preview 1](/ruby/rails/turbolinks/assets/bonusly_demo1.gif)

## But That Looks Like Shit
Agreed, so let's make it look a bit better:

(in _custom_styles.html.erb)
```scss
.turbolinks-progress-bar {
  height: 5px;
  background-color: <%= theme[:brand] %>;
}
```

Now it looks slightly less shitty:
![Bonusly Preview 2](/ruby/rails/turbolinks/assets/bonusly_demo2.gif)
![Bonusly Preview 3](/ruby/rails/turbolinks/assets/bonusly_demo3.gif)

## But Why?
#### 1. It feels faster

Normal requests usually flash to a white screen when the DOM is unloaded.  Not only is this jarring, but it leaves the user staring at nothing while the page loads.  This is bad.  Compare these two gifs:
![Bonusly without turbolinks](/ruby/rails/turbolinks/assets/bonusly_white_1.gif)
![Bonusly with turbolinks](/ruby/rails/turbolinks/assets/bonusly_white_2.gif)
_Note that both of these are running locally and throttled to the same speed, so there's no shenanigans going on here._

#### 2. It is faster
The network graph should speak for itself:
![Bonusly without turbolinks](/ruby/rails/turbolinks/assets/bonusly_network_without.png)
![Bonusly with turbolinks](/ruby/rails/turbolinks/assets/bonusly_network_with.png)

This is because Turbolinks does not reload javascript, images, or styles.  While this is usually not an issue, it can cause unexpected behavior when your Javascript expects a clean state.  Likewise, if your styles are dynamically generated (like our _custom_styles.html.erb), it will not reload the styles between requests unless you explicitly tell it to.

#### 3. It's good UX
The progress bar that ships with Turbolinks fulfills one of the core values of good UX: [visibility of system status](https://uxplanet.org/golden-rules-of-user-interface-design-19282aeb06b#b93a).  Users should always know what is going on (or at least think they do) because it both keeps them engaged and reduces frustrations.  If a user clicks a link and nothing happens, they're going to be much more frustrated than if they see a spinning wheel or a progress bar inching it's way across the screen.

It may not be useful from a pragmatic standpoint, but it gives the illusion of control and keeps the user engaged.

## Problems
The main issues that arise from Turbolinks are a result of not reloading static files and not refreshing the dom:

#### 1. CSS is not reloaded.
For example, our `_custom_styles.html.erb` file: ![Custom Style Horror](/ruby/rails/turbolinks/assets/custom_styles.gif)

#### 2. Event handlers are not cleaned up, resulting in them being bound twice.
For example, the javascript-based package manager: ![Event Horrors](/ruby/rails/turbolinks/assets/event_handlers.gif)

#### 3. Capybara is not happy.
Since Turbolinks doesn't actually load a new page, Capybara thinks that clicking a link is the end of the request.  In order to get them to work nicely, you'll need to wait for ajax after _every_ click event.  To combat this, it's often recommended that the `default_wait_time` is increased.

Could probably monkey-patch the click method, but definitely not ideal.

