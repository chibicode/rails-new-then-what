# rails-new-then-what

What I usually do after running "rails new", so I don't forget.

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

- `figaro` is helpful for setting config on Heroku.

```ruby
ruby "<ruby-version>"

gem "puma"
gem "foreman"
gem "figaro"
gem "slim-rails"

group :production do
  gem "rails_12factor"
end
```

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

Also for some reason, when using Postgres.app version 9.4, I need to uncomment this line on `database.yml`:

```yml
host: localhost
```

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

### Add Development/Test Gems

Example:

```ruby
group :development, :test do
  gem "web-console", "~> 2.0"
  gem "spring"
  gem "spring-commands-rspec"
  gem "rspec-rails"
  gem "jazz_fingers"
  gem "priscilla"
end

group :development do
  gem "quiet_assets"
  gem "annotate"
  gem "letter_opener"
  gem "bullet"
end

group :test do
  gem "factory_girl_rails"
  gem "database_cleaner"
end
```

### Configure Rspec

TBD

### Configure Node

TBD

Note to self: `npm run browser-sync ...`
