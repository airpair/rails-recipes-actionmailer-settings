### Intro

When Rails was released the core team proclaimed that now you will be free of configurations. Rails using Conventions-over-Configurations design paradigm and it looks really great when you are tired of tons of configuration files.

The core team reached the top of this paradigm when they left only one configuration file for rails application. I'm talking about `database.yml`. Actually, you don't have configure anything else except `database.yml` file before the application is started.

For a long time Rails have been used this technical feature to attract beginners. And I think it is an awesome promotional strategy for Rails. 

But I think you know in real development process we often have problems with application settings. It happens because Rails doesn't have default configuration management system.

If you have been following for evolution of Rails you have to know about new configuration file in Rails application: `secrets.yml`. And I think it is yet another proof that Rails feels discomfort because of the lack of default configuration management system.

Need more? For example, few my projects uses `autoprefixer.yml`, `new_relic.yml`.

What's next, Rails?

Certainly, Rails provides initializers, but I want to ask "Why I should to write ruby code when I just want to set API keys for one more web service?"

### The Problem

Configuration of ActionMailer is really big problem for many Rails developers, especially for Rails beginners.

I have asked many developers:

**"How did you configure your ActionMailer? Where and how do you keep your settings?"**

Some developers said

**"We use `config/environments/production.rb` or `config/environments/developemnt.rb` files"**

Some developers said

**"We use `config/application.rb` file"**

Usually it looks like

```ruby,linenumbers=true
  config.action_mailer.default_url_options = { host: 'example.com' }
  config.action_mailer.perform_deliveries    = true
  config.action_mailer.raise_delivery_errors = true
  config.action_mailer.delivery_method = :sendmail
  config.action_mailer.sendmail_settings = {
    location: '/usr/sbin/sendmail',
    arguments: '-i'
  }
```

So, it's nothing special, isn't it? Everybody does that!

Maybe I'm wrong, but I think it's totally bad practice. Why?

Every time when you want to change you ActionMailer settings you have to change code of your app. What if you hade one code for few projects? What would you do? Would you add conditions in your code?

That's why we should to find another way to configure our Rails application.

### Do it in a different way!

So, now I'll show you my recipe how to configure ActionMailer, which helps me do it fast and uniformly.

First of all we should select any configuration management system for Rails. I recommend you `gem "rails_config"` or `gem 'settingslogic'`. I will use **rails_config**.

#### Step 1
Add `gem "rails_config"` into your `Gemfile`

#### Step 2
`bundle install`

#### Step 3
Setup **rails_config** with

`rails g rails_config:install`

This will generate **config/initializers/rails_config.rb** with a set of default settings as well as to generate a set of default settings files:
```
config/settings.yml
config/settings/development.yml
config/settings/production.yml
config/settings/test.yml
```

Now __we have settings files for each environment__. Awesome!

#### Step 4
Put the following YML settings into **config/settings/production.yml** and **config/settings/development.yml**.

```yaml
mailer:
  service: smtp

  host: localhost:3000

  sandmail:
    location:  '/usr/sbin/sendmail'
    arguments: '-i -t'

  smtp:
    address: 'smtp.gmail.com'
    domain:  'gmail.com'
    port:    587

    email:   'test-account@gmail.com'
    password: QWERTY_PASSWORD
    authentication: plain
```

By means if line `service: smtp` you can select required configuration section for your ActionMailer. You can use the following values: `smtp`, `sandmail` or `test`.

#### Step 5
Finally, add code for ActionMailer configuration into **config/application.rb**. Your **config/application.rb** should look like this.

```ruby
module MyRailsApplication
  class Application < Rails::Application

  # ======================================================
  #  Mailer settings
  # ======================================================
  config.action_mailer.default_url_options = { host: Settings.mailer.host }

  if Settings.mailer.service == 'smtp'
    config.action_mailer.delivery_method = :smtp
    config.action_mailer.raise_delivery_errors = true

    config.action_mailer.smtp_settings = {
      address: Settings.mailer.smtp.address,
      domain:  Settings.mailer.smtp.domain,
      port:    Settings.mailer.smtp.port,

      user_name: Settings.mailer.smtp.email,
      password:  Settings.mailer.smtp.password,

      authentication: Settings.mailer.smtp.authentication,
      enable_starttls_auto: true
    }
  elsif Settings.mailer.service == 'sandmail'
    config.action_mailer.raise_delivery_errors = true

    config.action_mailer.sendmail_settings = {
      location:  Settings.mailer.sandmail.location,
      arguments: Settings.mailer.sandmail.arguments
    }
  else
    config.action_mailer.delivery_method = :test
    config.action_mailer.raise_delivery_errors = false
  end
  # ======================================================
  #  ~ Mailer settings
  # ======================================================

end
```

That's all!

### Life after changes

0. Now you don't have change your application code when you need to change ActionMailer settings
0. You don't have deploy your code to production server when you need to change ActionMailer settings. Now, you can execute only `cap production config:update` instead `cap production deploy`
0. You can use your application code for different projects with different ActionMailer settings.

### Conclusion

By this article I tried to show my own recipe to configure ActoinMailer in Rails app. I will be glad to hear your opinion about it. Where and how do you keep your settings? Which approach do you use for ActionMailer configuration? Let's discuss it!

**P.S.**
You can subscribe to my mailing list if you want know about my new articles and gists: <a href='http://eepurl.com/bh6nZ5'>Mailing List: Articles for Developers</a> or follow me on github.com: <a href="https://github.com/the-teacher">github.com/the-teacher</a>