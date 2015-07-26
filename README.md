![](pablo.png)

I prefer documenting bootstrap steps over creating templates/automation scripts. Why? Because sometimes I might want to customize an app generated from a script, but without documentation I'd have no idea how to customize it. But if there's documentation on how to bootstrap apps, there's often no need for automations scripts - just follow the steps! No need to keep documentation and scripts up to date with each other.

The downside is that sometimes the steps can take long and you might forget to perform some steps.

### Make sure Ruby/Rails are up to date

```sh
# rbenv and ruby-build are installed from git, not from brew
cd ~/.rbenv && git pull
cd ~/.rbenv/plugins/ruby-build && git pull

# Also installs gems on ~/.rbenv/default-gems
rbenv install <latest-ruby-version>

gem update rails # or gem install rails
```

Here's my `.railsrc`:

```sh
-d postgresql -T
```

### Create a GitHub Repo

I use GitHub for Mac, so I just add the folder - GitHub for Mac takes care of creating a repo locally and on GitHub.

### Create a Heroku App

I usually create the app on Heroku, and then do:

```sh
heroku git:remote -a <app-name>
```

### Create a Tmuxinator Configuration

Useful to keep a server up and runnning.

`atom ~/.tmuxinator` and add `<app-name>.yml`:

```yml
name: <app-name>
root: ~/code/<app-name>
windows:
  - shell: ""
  - shell: ""
  - shell: ""
  - shell: ""
  - shell: ""
  - shell: ""
  - shell: ""
```

### Add Root-Level Files

Copy files under the [src](src) directory. Optionally modify `.ruby-version` and `.env`.

Optionally, copy `.rubocop.yml`, `.scss-lint.yml`, etc from the root.

### Edit Gemfile

Change ruby version.

### Initialize Puma

Add `config/puma.rb` and copy the contents from [this page](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#config).

#### :warning: Set # of workers to 1

Set the number of workers to 1 locally so that `web-console` works (see [here](https://github.com/rails/web-console/pull/109) and [here](https://github.com/charliesome/better_errors#unicorn-puma-and-other-multi-worker-servers))

```
workers Integer(ENV['WEB_CONCURRENCY'] || 1)
```

### Create Database

```sh
rake db:create
```

I also try to add `<app-name>_development` to [Postico.app](https://eggerapps.at/postico/).

:warning: Also for some reason, when using Postgres.app version 9.4, I need to uncomment this line on `database.yml`:

```yml
host: localhost
```

AND add the above line under `test:` as well.

### Add Root Controller

- Create `HomeController`
- Set root route
- Add `views/home/index.slim`

At this point,

```
foreman start
```

and accessing [http://localhost:5000](http://localhost:5000) should give you a blank page.

### Edit `application.rb`

Set time zone:

```ruby
config.time_zone = "Pacific Time (US & Canada)"
```

Set schema format:

```ruby
config.active_record.schema_format = :sql
```

Then run `rake db:migrate` to create `structure.sql`.

### Configure Rspec

Run the initializer:

```sh
rails g rspec:install
```

Generate binstubs:

```sh
bundle exec spring binstub rspec
```

Some things you should add to `spec_helper.rb`:

```ruby
require "active_support/all"
```

Some things you should add to `rails_helper.rb`:

```ruby
require 'factory_girl_rails'
require 'database_cleaner'

# Uncomment:
Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }

# Remove the line on `fixture_path`, and set transactional fixtures to false:
config.use_transactional_fixtures = true
```

Add `spec/support/configure_factory_girl.rb`:

```ruby
RSpec.configure do |config|
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, js: true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end

  config.include FactoryGirl::Syntax::Methods
end
```

Test that this works by creating `spec/controllers/application_controller_spec.rb`:

```ruby
require "rails_helper"

RSpec.describe ApplicationController do
  it "works" do
  end
end
```

### Configure Webpack (Replace Asset Pipeline)

The basic [package.json](package.json) is provided. You then need to set up BrowserSync and Webpack. Use the configs from [this repo](http://github.com/chibicode/webpack-notes).

Add `frontend/dist`, the target for webpack, to path on `config/initializers/assets.rb`:

```ruby
Rails.application.config.assets.paths << Rails.root.join("frontend", "dist")
```

Modify `application.js` to use the generated file:

```
// require frontend
```

Remove CSS import from `application.html.erb`:

```erb
<%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
```

### Configure BrowserSync

Generate the script:

```
browser-sync init
```

Set up files to watch:

```
files": [""app/views/**/*"],
```

Proxy the port:

```
"proxy": "localhost:5000"
"port": 3002,
```

### Configure Development Procfile

Create `bin/server` and `chmod +x bin/server`:

```sh
#!/bin/sh

foreman start -f Procfile.dev
```

Create `Procfile.dev`:

```
web: bundle exec puma -C config/puma.rb
webpack: $(npm bin)/webpack-dev-server --inline
browsersync: $(npm bin)/browser-sync --config bs-config.js start
```

### Add Services

Here are some of my favorite addons:

- Continuous Deployment: [CodeShip](http://codeship.io/)
- Bug Tracking: [BugSnag](https://bugsnag.com/)
- Logging: [LogEntries](https://logentries.com/)
- Performance Tracking: [Skylight](https://www.skylight.io/)
- Background Queues: [Sidekick](http://sidekiq.org/) with [Heroku Redis](https://elements.heroku.com/addons/heroku-redis)
- Fragment Caching Backend: [Memcachier](https://www.memcachier.com/)
