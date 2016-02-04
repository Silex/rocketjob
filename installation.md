---
layout: default
---

## Installation

[Rocket Job][0] can run with or without Rails. Instructions for Rails and Standalone installations are listed below.

### Compatibility

* Ruby 1.9.3, 2.0, 2.1, 2.2, 2.3, or greater
* JRuby 1.7, 9.0.4.0, or greater
* [MongoDB][3] V2.6 or greater is required.

### MongoDB

[Rocket Job][0] stores jobs in the open source data store [MongoDB][3].
Installing [MongoDB][3] is relatively straight forward.

For example, installing [MongoDB][3] on a Mac running homebrew:

~~~
brew install mongodb
~~~

Then follow the on-screen instructions to start [MongoDB][3].

For other platforms, see [MongoDB Downloads](https://www.mongodb.org/downloads)

## Rails Installation

For an existing Rails installation, add the following lines to the bottom of the file `Gemfile`:

~~~ruby
gem 'rocketjob'
gem 'bson_ext', platform: :ruby
gem 'rails_semantic_logger'
~~~

Install gems:

~~~
bundle install
~~~

Generate Mongo Configuration file:

~~~
bundle exec rails generate mongo_mapper:config
~~~

If you want to configure your application with a MongoDB URI (i.e. on Heroku), then you can use
the following settings for your production environment in `config/mongo.yml`:

~~~yaml
production:
 uri: <%= ENV['MONGODB_URI'] %>
~~~

If you are running `Spring`, which is installed by default by Rails, stop the backgound
spring processes to get them to reload:

~~~
bin/spring stop
~~~

Start a Rocket Job worker process:

~~~
bundle exec rocketjob
~~~

Or, if you have generated bundler bin stubs:

~~~
bin/rocketjob
~~~

### Installing RocketJob Mission Control (Web Interface)

[Rocket Job Mission Control][1] is the web interface for [Rocket Job][0].
It is a rails engine that can be added to any existing Rails 4 or Rails 5 rails application.

Add the [Rocket Job Mission Control][1] gem to your Gemfile

~~~ruby
gem 'rocketjob_mission_control'
~~~

Install gems:

~~~
bundle install
~~~

Add the following line to `config/routes.rb` in your Rails application:

~~~ruby
mount RocketJobMissionControl::Engine => 'rocketjob'
~~~

Start the Rails server:

~~~
bin/rails s
~~~

Open a browser and navigate to the local [Rocket Job Mission Control](http://localhost:3000/rocketjob)

## Standalone Installation

When running stand-alone without Rails.

Create directories to hold the standalone RocketJob jobs and configuration:

~~~
mkdir standalone
mkdir standalone/jobs
mkdir standalone/config
cd standalone
~~~

Create a file called `Gemfile` in the `standalone` directory with the following contents:

~~~ruby
source 'https://rubygems.org'

gem 'rocketjob'
gem 'bson_ext', platform: :ruby
~~~

Install the gem files:

~~~
bundle
~~~

Create a file called `mongo.yml` in the `config` sub-directory with the following contents:

~~~yaml
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
~~~

Create a new Job for the workers to process. Create a file called `hello_world_job.rb`
in the `jobs` directory with the following contents:

~~~ruby
class HelloWorldJob < RocketJob::Job
  def perform
    puts "HELLO WORLD"
  end
end
~~~

Start a worker process, from within the `standalone` directory:

~~~
bundle exec rocketjob
~~~

Open a new console to queue a new job request:

~~~
bundle exec irb
~~~

Enter the following code:

~~~ruby
require 'rocketjob'

# Log to development.log
SemanticLogger.add_appender('development.log', &SemanticLogger::Appender::Base.colorized_formatter)
SemanticLogger.default_level = :debug

# Configure Mongo
RocketJob::CLI.load_config('development', 'config/mongo.yml')

require_relative 'jobs/hello_world_job'

HelloWorldJob.create!
~~~

The console running `rocketjob` should show something similar to:

~~~
2015-09-09 22:54:41.979435 I [25215:rocketjob 1] [Job 55f0f0f1a26ec06280000001] HelloWorldJob -- Start HelloWorldJob#perform
HELLO WORLD
2015-09-09 22:54:41.979662 I [25215:rocketjob 1] [Job 55f0f0f1a26ec06280000001] (0.1ms) HelloWorldJob -- Completed HelloWorldJob#perform
~~~

### Installing RocketJob Mission Control (Web Interface)

[Rocket Job Mission Control][1] is the web interface for [Rocket Job][0].
In order to install [Rocket Job Mission Control][1] in a stand-alone environment, we need to
host it in a "shell" rails application as follows:

Create shell application:

~~~
gem install rails
rails new rjmc
cd rjmc
~~~

Add the following lines to the bottom of the file `Gemfile`:

~~~ruby
gem 'rocketjob'
gem 'bson_ext', platform: :ruby
gem 'rails_semantic_logger'
gem 'rocketjob_mission_control'
~~~

Now run `bundle` to install [Rocket Job Mission Control][1]

Add the following line to `config/routes.rb`:

~~~ruby
mount RocketJobMissionControl::Engine => 'rocketjob'
~~~

Re-load spring:

~~~
bin/spring stop
~~~

Start the stand-alone [Rocket Job Mission Control][1]:

~~~
bin/rails s
~~~

Open a browser and navigate to the local [Rocket Job Mission Control](http://localhost:3000/rocketjob)

### [Next: Guide ==>](guide.html)

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
