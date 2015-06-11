# Stormpath-Rails-Gem

Stormpath is the first easy, secure user management and authentication service for developers. This is the Rails gem to ease integration of its features with any Rails-based application.

stormpath makes it incredibly simple to add users and user data to your application. It aims to completely abstract away all user registration, login, authentication, and authorization problems, and make building secure websites painless.

## INSTALL

To get started, add Stormpath to your `Gemfile`, `bundle install`, and run the
`install generator`:

generates inital config and setup files
```sh
$ rails generate stormpath:install
```

The generator:

* Inserts `Stormpath::Controller` into your `ApplicationController`
* Creates an initializer to allow further configuration.
* Creates a migration that either creates a users table or adds any necessary
  columns to the existing table.

configure MODEL to use Stormpath
```sh
rails generate stormpath MODEL
```

Replace MODEL with the class name used for the application’s users. It’s frequently User but could also be anything else.

# CONFIGURE
Override any of these defaults in config/initializers/stormpath.rb

```ruby
Stormpath.configure do |config|
  config.api_key_file = '/Users/robert/.stormpath/apiKey.properties'
  config.application = 'https://api.stormpath.com/v1/applications/xxx'
  config.secret_key = 'some_long_random_string'
  congig.expand_custom_data = true
  config.enable_forgot_password = true
  config.id_site = false
  config.mailer_sender = 'reply@example.com'
end
```

# USAGE

### Access Control
Use the `require_login` to control access to controller actions
```ruby
class ArticlesController < ApplicationController
  before_action :require_login

  def index
  end
end
```

Stormpath provides routing constraints that can be used to control access at the routing layer

```ruby
Blog::Application.routes.draw do
  constraints Stormpath::Constraints::SignedIn.new { |user| user.admin? } do
    root to: 'admin/dashboards#show', as: :admin_root
  end

  constraints Stormpath::Constraints::SignedIn.new do
    root to: 'dashboards#show', as: :signed_in_root
  end

  constraints Stormpath::Constraints::SignedOut.new do
    root to: 'marketing#index'
  end
end
```

### Helper Methods
you can access user session for this scope:
```ruby
user_session
```

Use `current_user`, `signed_in?` in controllers, views, and helpers. For example:
```erb
<% if signed_in? %>
  <%= current_user.email %>
  <%= button_to "Sign out", sign_out_path, method: :delete %>
<% else %>
  <%= link_to "Sign in", sign_in_path %>
<% end %>
```

## Overriding Stormpath

### Routes
You can optionally run `rails generate stormpath:routes` to dump a copy of the default routes into your application for modification

```sh
rails generate stormpath:routes
```

### Controllers
To override a Stormpath controller, subclass it and update the routes to point to your new controller (see the "Routes" section).
```ruby
class PasswordsController < Stormpath::PasswordsController
class SessionsController < Stormpath::SessionsController
class UsersController < Stormpath::UsersController
```

### Redirects
All of these controller methods redirect to `'/'` by default:
```
passwords#url_after_update
sessions#url_after_create
sessions#url_for_signed_in_users
users#url_after_create
application#url_after_denied_access_when_signed_in
application#url_after_denied_access_when_signed_out
```

### Views
You can use the stormpath views generator to copy the default views to your application for modification.
```sh
rails generate stormpath:views
```

```
app/views/stormpath_mailer/change_password.html.erb
app/views/passwords/create.html.erb
app/views/passwords/edit.html.erb
app/views/passwords/new.html.erb
app/views/sessions/_form.html.erb
app/views/sessions/new.html.erb
app/views/users/_form.html.erb
app/views/users/new.html.erb
```

if you would like to generate just only a few set of views, like the sessions you can pass a list of modules to the generator with the -v flag
```sh
rails generate stormpath:views -v sessions
```

### Translations
All flash messages and email subject lines are stored in [i18n translations]
(http://guides.rubyonrails.org/i18n.html). Override them like any other
translation.
