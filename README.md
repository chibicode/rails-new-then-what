# rails-new-then-what

What I usually do after running "rails new", so I don't forget.

I prefer documenting bootstrap steps over creating templates/automation scripts. Why? Because sometimes I might want to customize an app generated from a script, but without documentation I'd have no idea how to customize it. But if there's documentation on how to bootstrap apps, there's often no need for automations scripts - just follow the steps! No need to keep documentation and scripts up to date with each other.

The downside is that sometimes the steps can take long and you might forget to perform some steps.

### Keep This Document Updated

Once in a while, update linter configs for Atom:


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
