# A logger for Grape apps
[![Code Climate](https://codeclimate.com/github/ridiculous/grape-middleware-logger/badges/gpa.svg)](https://codeclimate.com/github/ridiculous/grape-middleware-logger) [![Gem Version](https://badge.fury.io/rb/grape-middleware-logger.svg)](http://badge.fury.io/rb/grape-middleware-logger)
[![Build Status](https://travis-ci.org/ridiculous/grape-middleware-logger.svg)](https://travis-ci.org/ridiculous/grape-middleware-logger)

Logs:
  * Request path
  * Parameters
  * Endpoint class name and handler
  * Response status
  * Duration of the request
  * Exceptions
  * Error responses from `error!`

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'grape', '>= 0.14.0'
gem 'grape-middleware-logger'
```

## Usage
```ruby
class API < Grape::API
  # @note Make sure this above you're first +mount+
  use Grape::Middleware::Logger
end
```

Server requests will be logged to STDOUT by default.

## Custom setup
Customize the logging by passing the `logger` option. Example using a CustomLogger and parameter sanitization:
```ruby
use Grape::Middleware::Logger, {
  logger: CustomLogger.new,
  filter: CustomFilter.new
}
```
The `logger` option can be any object that responds to `.info(msg)`

The `filter` option can be any object that responds to `.filter(params_hash)`

## Example output
Get
```
Started GET "/v1/reports/101" at 2015-12-11 15:40:51 -0800
Processing by ReportsAPI#reports/:id
  Parameters: {"id"=>"101"}
Completed 200 in 6.29ms
```
Error
```
Started POST "/v1/reports" at 2015-12-11 15:42:33 -0800
Processing by ReportsAPI#reports
  Parameters: {"name"=>"foo", "password"=>"[FILTERED]"}
  Error: {:error=>"undefined something something bad", :detail=>"Whoops"}
Completed 422 in 6.29ms
```

## Using Rails?
`Rails.logger` and `Rails.application.config.filter_parameters` will be used automatically as the default logger and 
param filterer, respectively.

You may want to disable Rails logging for API endpoints, so that the logging doesn't double-up. You can achieve this 
by switching around some middleware. For example:

```ruby
# config/application.rb
config.middleware.delete 'Rails::Rack::Logger'
config.middleware.insert_after 'ActionDispatch::RequestId', 'SelectiveLogger'

# config/initializers/selective_logger.rb
class SelectiveLogger
  def initialize(app)
    @app = app
  end

  def call(env)
    if env['PATH_INFO'] =~ %r{^/api}
      @app.call(env)
    else
      Rails::Rack::Logger.new(@app).call(env)
    end
  end
end
```

## Rack

If you're using the `rackup` command to run your server in development, pass the `-q` flag to silence the default rack logger.

## Credits

Big thanks to jadent's question/answer on [stackoverflow](http://stackoverflow.com/questions/25048163/grape-using-error-and-grapemiddleware-after-callback)
for easily logging error responses. Borrowed some motivation from the [grape_logging](https://github.com/aserafin/grape_logging) gem
and would love to see these two consolidated at some point.

## Contributing

1. Fork it ( https://github.com/ridiculous/grape-middleware-logger/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
