# ElasticNotifier

[![Maintainability](https://api.codeclimate.com/v1/badges/92763b5c5c012431d829/maintainability)](https://codeclimate.com/github/hspazio/elastic_notifier/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/92763b5c5c012431d829/test_coverage)](https://codeclimate.com/github/hspazio/elastic_notifier/test_coverage)

ElesticNotifier gem provides a simple API to send error notifications to an Elastic Search instance. 

It also comes with a built-in plugin for [exception_notification][exception_notification] gem to send error notifications caught by the Rack middleware. 

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'elastic_notifier'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install elastic_notifier

## As a standalone notifier

Use the code below if you want to override the default configurations. For __Rails__ applications you can add the snippet to `config/initializers/elastic_notifier.rb`

```ruby
ElasticNotifier.configure do |config|
  # default url from environment variable: ENV['ELASTIC_NOTIFIER_URL']
  config.url = 'http://myelasticsearch.server.com:9200' 
  config.index = :my_custom_index  # default is :elastic_notifier
  config.type = :my_document_type  # default is :signals
end
```
Then send error notifications as you rescue errors:

```ruby
begin
  # some code that raises an exception
rescue => error
  ElasticNotifier.notify_error(error)
end
```

## As ExceptionNotification's Plugin

If you are already using [exception_notification][exception_notification] gem within a Rails app you can use ElasticNotifier's plugin.

Simply add the configurations to the `ExceptionNotification::Rack` middleware [as documented here](https://github.com/smartinez87/exception_notification#rails).

```ruby
Rails.application.config.middleware.use ExceptionNotification::Rack,
  :email => {
    :email_prefix => "[PREFIX] ",
    :sender_address => %{"notifier" <notifier@example.com>},
    :exception_recipients => %w{exceptions@example.com}
  },
  :elastic_search => {
    :url => 'http://myelasticsearch.server.com:9200',
    :index => 'elastic_notifier',
    :username => 'signals'
  }
```

## What information is being sent?

At the time the notifier is invoked it collects some information from the environment, serializes it together with the exception details and sent it to the Elastic instance.

```ruby
{
  severity: "error",
  timestamp: "2017-12-31 23:59:59",
  program_name: "my_app.rb",
  pid: 1345,
  hostname: "myservicename",
  ip: "123.123.123.123",
  data: {
    name: "NoMethodError",
    message: "undefined method `test` for nil:NilClass",
    backtrace: [...]
  }
}

```

## Contributing

Bug reports and pull requests are very welcome. Please be aware that you are expected to follow the [code of conduct](https://github.com/hspazio/elastic_notifier/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright (c) 2017 Fabio Pitino, released under the [MIT license](http://www.opensource.org/licenses/MIT).

[exception_notification]: https://github.com/smartinez87/exception_notification
