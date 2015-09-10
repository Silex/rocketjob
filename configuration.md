---
layout: default
---

## Configuration

## New Rails installation

Create a new Rails application, if one does not exist already

```
gem install rails
rails new myapp
cd myapp
```

## Configure existing Rails application

Add the following lines to the bottom of the file `Gemfile`:

```ruby
gem 'rocketjob'
gem 'bson_ext', platform: :ruby
gem 'rails_semantic_logger'
```

Generate Mongo Configuration file:

```
bundle exec rails generate mongo_mapper:config
```

If you are running `Spring`, which is installed by default by Rails, stop the backgound
spring processes to get them to reload:

```
bin/spring stop
```

Start a rocketjob worker process:

```
bundle exec rocketjob
```

Or, if you have generated bundler bin stubs:

```
bin/rocketjob
```

If you want to configure your application with a MongoDB URI (i.e. on Heroku), then you can use the following settings for your production environment:

```yaml
production:
 uri: <%= ENV['MONGODB_URI'] %>
```


# Standalone Configuration

When running stand-alone without Rails.

Create a new directory for running RocketJob workers:

```
mkdir standalone
mkdir standalone/jobs
mkdir standalone/config
cd standalone/config
```

Create a `mongo.yml` configuration file with the following contents:

```yaml
defaults: &defaults
  host: 127.0.0.1
  port: 27017
  options:
    w: 1
    pool_size: 1
    slave_ok: false
    ssl: false

development:
  <<: *defaults
  database: myapp_development

test:
  <<: *defaults
  database: myapp_test
  w: 0

# set these environment variables on your prod server
production:
  <<: *defaults
  database: myapp
  username: <%= ENV['MONGO_USERNAME'] %>
  password: <%= ENV['MONGO_PASSWORD'] %>
```

Change back to the standalone directory

```
cd ..
```

Create a file called `Gemfile` with the following contents:

```ruby
gem 'rocketjob'
gem 'bson_ext', platform: :ruby
```

Install the gem files:

```
bundle
```


Create a new Job for the workers to process. Create a file called `hello_world_job.rb`
in the `jobs` directory with the following contents:

```ruby
class HelloWorldJob < RocketJob::Job
  def perform
    puts "HELLO WORLD"
  end
end
```

Start a worker process, from within the `standalone` directory:

```
bundle exec rocketjob
```

Open a new console to queue a new job request:

```
bundle exec irb
```

Enter the following code:

```ruby
require 'rocketjob'

# Log to development.log
SemanticLogger.add_appender('development.log', &SemanticLogger::Appender::Base.colorized_formatter)
SemanticLogger.default_level = :debug

# Configure Mongo
RocketJob::CLI.load_config('development', 'config/mongo.yml')

require_relative 'jobs/hello_world_job'

HelloWorldJob.create!
```

The console running `rocketjob` should show something similar to:

```
2015-09-09 22:54:41.979435 I [25215:rocketjob 1] [Job 55f0f0f1a26ec06280000001] HelloWorldJob -- Start HelloWorldJob#perform
HELLO WORLD
2015-09-09 22:54:41.979662 I [25215:rocketjob 1] [Job 55f0f0f1a26ec06280000001] (0.1ms) HelloWorldJob -- Completed HelloWorldJob#perform
```

## [Next: Architecture ==>](architecture.html)
