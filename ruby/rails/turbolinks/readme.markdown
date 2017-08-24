# [Turbolinks](https://github.com/turbolinks/turbolinks)
Ever noticed those little loading bars at the top of some websites?

Youtube:
![Youtube Preview](/ruby/rails/turbolinks/assets/youtube_demo.gif)
Medium:
![Medium Preview](/ruby/rails/turbolinks/assets/medium_demo.gif)
Github:
![Github Preview](/ruby/rails/turbolinks/assets/github_demo.gif)

While they may not be using the Turbolinks library, they're leveraging the same principle of changing the page content without fully refreshing the DOM.

# Overview
_NOTE: This is basically just an abbreviated version of [the turbolinks readme](https://github.com/turbolinks/turbolinks).  If this is interesting to you, you're better off reading that._

Turbolinks is a way of giving a website all of the benefits of a single-page application without the overhead of a client-side Javascript framework.  The library allows the backend to remain unchanged while relying on javascript to intercept click events and render the page without incurring a full page load.  It'll be default in Rails 5 so we'd better get used to it now. :dancer:

Honestly just go to the Turbolinks repo if you want to read things, I don't word good.

# But how?
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

# But that looks like shit
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

# But why?
#### 1. It feels faster

Normal requests usually flash to a white screen when the DOM is unloaded.  Not only is this jarring, but it leaves the user staring at nothing while the page loads.  This is bad.  Compare these two gifs:
![Bonusly without turbolinks](/ruby/rails/turbolinks/assets/bonusly_white_1.gif)
![Bonusly with turbolinks](/ruby/rails/turbolinks/assets/bonusly_white_2.gif)
_Note that both of these are running locally and throttled to the same speed, so there's no shenanigans going on here._

Turbolinks also renders cached versions of pages you have visited before _until_ it has downloaded the new content, then it shows the newest content.  This makes it _look_ like the page is loading faster than it actually it.

#### 2. It is faster
The network graph should speak for itself:

Without Turbolinks: ![Bonusly without turbolinks](/ruby/rails/turbolinks/assets/bonusly_network_without.png)
With Turbolinks: ![Bonusly with turbolinks](/ruby/rails/turbolinks/assets/bonusly_network_with.png)

This is because Turbolinks does not reload javascript, images, or styles.  While this is usually not an issue, it can cause unexpected behavior when your Javascript expects a clean state.  Likewise, if your styles are dynamically generated (like our _custom_styles.html.erb), it will not reload the styles between requests unless you explicitly tell it to.

#### 3. It's good UX
The progress bar that ships with Turbolinks fulfills one of the core values of good UX: [visibility of system status](https://uxplanet.org/golden-rules-of-user-interface-design-19282aeb06b#b93a).  Users should always know what is going on (or at least think they do) because it both keeps them engaged and reduces frustrations.  If a user clicks a link and nothing happens, they're going to be much more frustrated than if they see a spinning wheel or a progress bar inching it's way across the screen.

It may not be useful from a pragmatic standpoint, but it gives the illusion of control and keeps the user engaged.

# Problems & Solutions
The main issues that arise from Turbolinks are a result of not reloading static files and not refreshing the dom:

#### 1. CSS is not reloaded.
For example, our `_custom_styles.html.erb` file: ![Custom Style Horror](/ruby/rails/turbolinks/assets/custom_styles.gif)
The solution to this is to add the data attribute `data-turbolinks-track` to all of the static assets you want to reload, like so:
```html
<head>
    <link rel="stylesheet" href="/custom_styles.css" data-turbolinks-track="reload" />
    <script src="/custom_page_scripts.css" data-turbolinks-track="reload" />
</head>
```

#### 2. Event handlers are not cleaned up, resulting in them being bound twice.
For example, the javascript-based package manager: ![Event Horrors](/ruby/rails/turbolinks/assets/event_handlers.gif)

This is actually one of the most difficult issues to fix because most libraries don't unbind events before binding them (because why would you?).  Having to trace down the source of each of these hard to discover bugs and then add functionality to unbind the events on each page load is definitely not ideal, but it's the recommended solution for most cases.

Luckily this issue seems to only pop up when the JS is inline instead of externally loaded, and in those cases you can add the `data-turbolinks-eval='false'` attribute to your script tag to prevent it from firing after rendering.

#### 3. Capybara is not happy.
Since Turbolinks doesn't actually load a new page, Capybara thinks that clicking a link is the end of the request.  In order to get them to work nicely, you'll need to wait for ajax after _every_ click event.  To combat this, it's often recommended that the `default_wait_time` is increased.

Could probably monkey-patch the click method, but definitely not ideal.

#### 4. Redirects are poorly respected
Turbolinks uses `XMLHttpRequest`, which follows redirects silently.  This means that there's no way for Turbolinks to know that the page has been redirected, so it will show the __original__ url in the address bar instead of the correct, redirected url.  Luckily the rails engine automatically handles this by adding the `Turbolinks-Location` header when `redirect_to` is called.  But if you redirect using another method, you'll have to set the header manually.

#### 5. You'll need to change the way events are handled
Because the DOM is no longer reloaded and the page doesn't actually change, `window.onload`, `DOMContentLoaded`, and `$(document).ready()` events no longer work as expected.  Instead, you'll need to listen to the `turbolinks:load` event, which fires once on the initial page load and again after every Turbolinks visit.  For example:
```javascript
document.addEventListener('turbolinks:load', function() {
  alert('Shucky ducky');
})
```
