# Marketing Engine

### What's an engine?
Engines can be considered miniature applications that provide functionality to their host applications. A Rails application is actually just a "supercharged" engine, with the Rails::Application class inheriting a lot of its behavior from Rails::Engine.

Devise is an example of an engine that we're all familiar with.

### Why'd we use an engine for marketing?
Since we want the marketing application to be as separate as
possible, using an engine allows us to basically start over
an only include the functionality that's absolutely
essential.

### Structure
```
special_sauce
|- app (This is where the main app lives)
|- apps (This is where the marketing app lives, and probably all future apps)
   |- marketing
      |- app
         |- assets
            |- components
               |- marketing
                  |- simple_sign_up
                     |- styles.scss
                     |- scripts.scss
            |- fonts
            |- images
            |- javascripts
               |- marketing
            |- stylesheets
               |- marketing
         |- controllers
            |- marketing
         |- helpers
            |- marketing
         |- views
            |- components
               |- marketing
            |- layouts
               |- marketing
            |- marketing
      |- config
      |- lib
      |- marketing.gemspec
```

The marketing app lives in the `apps` namespace.  (Nearly) Every file that exists in the marketing app should live under the
marketing module.  This ensures that we don't get naming conflicts between the main app and marketing.

### Mounting the routes
Add this to the main route file
```ruby
# in /config/routes.rb
mount Marketing::Engine => '/', as: 'marketing'
```

Then you can do this in the marketing routefile
```ruby
# in /apps/marketing/config/routes.rb
Marketing::Engine.routes.draw do
  root         controller: :home, action: :index, as: 'root_path'
  get '/home', controller: :home, action: :index, defaults: { force: true }

  scope :marketing do
    resources :component_library, only: %i[index]
  end
end
```

### The gemspec
Building the marketing engine as a gem allows us to separate dependencies.  Currently we throw all of our gems in the same file, but in
and ideal world we would only include the gems that are relevant to our current environment.  For example, if we only run the marketing
engine on a dyno, there's no reason to load most of the default gems.  If we continue with this pattern, we could fully separate
our dependencies and make it clearer what things are related.

Add this to gemfile:
```ruby
gemspec path: 'apps/marketing'
```
