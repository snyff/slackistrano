# Slackistrano

[![Gem Version](https://badge.fury.io/rb/slackistrano.png)](http://badge.fury.io/rb/slackistrano)
[![Code Climate](https://codeclimate.com/github/phallstrom/slackistrano.png)](https://codeclimate.com/github/phallstrom/slackistrano)
[![Build Status](https://travis-ci.org/phallstrom/slackistrano.png?branch=master)](https://travis-ci.org/phallstrom/slackistrano)

Send notifications to [Slack](https://slack.com) about [Capistrano](http://www.capistranorb.com) deployments.

**NOTE:** This README documents version >=3.1.0. You can read about 3.0.1 [here](https://github.com/phallstrom/slackistrano/tree/v3.0.1).

## Requirements

- Capistrano >= 3.5.0
- Ruby >= 2.0
- A Slack account

## Installation

1. Add this line to your application's Gemfile:

   ```ruby
   gem 'slackistrano'
   ```

2. Execute:

   ```
   $ bundle
   ```

3. Require the library in your application's Capfile:

   ```ruby
   require 'slackistrano/capistrano'
   ```

## Configuration

You have two options to notify a channel in Slack when you deploy:

1. Using *Incoming WebHooks* integration, offering more options but requires
   one of the five free integrations. This option provides more messaging
   flexibility.
2. Using *Slackbot*, which will not use one of the five free integrations.

### Incoming Webhook

1. Configure your Slack's Incoming Webhook.
2. Add the following to `config/deploy.rb`:

   ```ruby
   set :slackistrano, {
     channel: '#your-channel',
     webhook: 'your-incoming-webhook-url'
   }
   ```

### Slackbot

1. Configure your Slack's Slackbot (not Bot).
2. Add the following to `config/deploy.rb`:

   ```ruby
   set :slackistrano, {
     channel: '#your-channel',
     team: 'your-team-name',
     token: 'your-token'
   }
   ```

### Test your Configuration

Test your setup by running the following command. This will post each stage's
message to Slack in turn.

```
$ cap production slack:deploy:test
```

## Usage

Deploy your application like normal and you should see messages in the channel
you specified.

## Customizing the Messaging

You can customize the messaging posted to Slack by providing your own messaging
class and overriding several methods. Here is one example:

```ruby
module Slackistrano
  class CustomMessaging < Messaging::Base

    # Send failed message to #ops. Send all other messages to default channels.
    # The #ops channel must exist prior.
    def channels_for(action)
      if action == :failed
        "#ops"
      else
        super
      end
    end

    # 
    s updating message.
    def payload_for_updating
      nil
    end

    # Suppress reverting message.
    def payload_for_reverting
      nil
    end

    # Fancy updated message.
    # See https://api.slack.com/docs/message-attachments
    def payload_for_updated
      {
        attachments: [{
          color: 'good',
          title: 'Integrations Application Deployed :boom::bangbang:',
          fields: [{
            title: 'Environment',
            value: stage,
            short: true
          }, {
            title: 'Branch',
            value: branch,
            short: true
          }, {
            title: 'Deployer',
            value: deployer,
            short: true
          }, {
            title: 'Time',
            value: elapsed_time,
            short: true
          }],
          fallback: super[:text]
        }]
      }
    end

    # Default reverted message.  Alternatively simply do not redefine this
    # method.
    def payload_for_reverted
      super
    end

    # Slightly tweaked failed message.
    # See https://api.slack.com/docs/message-formatting
    def payload_for_failed
      payload = super
      payload[:text] = "OMG :fire: #{payload[:text]}"
      payload
    end

    # Override the deployer helper to pull the full name from the password file.
    # See https://github.com/phallstrom/slackistrano/blob/master/lib/slackistrano/messaging/helpers.rb
    def deployer
      Etc.getpwnam(ENV['USER']).gecos
    end
  end
end
```

The output would look like this:
![Custom Messaging](https://raw.githubusercontent.com/phallstrom/slackistrano/overhaul/images/custom_messaging.jpg)

To set this up:

1. Add the above class to your app, for example `lib/custom_messaging.rb`.

2. Require the library after the requiring of Slackistrano in your application's Capfile.

   ```ruby
   require_relative 'lib/custom_messaging'
   ```

3. Update the `slackistrano` configuration in `config/deploy.rb` and add the `klass` option.

   ```ruby
   set :slackistrano, {
     klass: Slackistrano::CustomMessaging,
     channel: '#your-channel',
     webhook: 'your-incoming-webhook-url'
   }
   ```

4. If you come up with something that you think others would enjoy submit it as
   an issue along with a screenshot of the output from `cap production
   slack:deploy:test` and I'll add it to the Wiki.

## TODO

- Notify about incorrect configuration settings.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
