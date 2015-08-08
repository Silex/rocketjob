---
layout: default
---

## Configuration

### Rails Configuration

MongoMapper will already configure itself in Rails environments. `rocketjob` can
be configured to use a separate MongoDB instance from the Rails application as follows:

For example, we may want `RocketJob::Job` to be stored in a Mongo Database that
is replicated across data centers, whereas we may not want to replicate the
`RocketJob::SlicedJob` slices due to the amount of network traffic it would generate.

```ruby
config.before_initialize do
  # Share the common mongo configuration file
  config_file = root.join('config', 'mongo.yml')
  if config_file.file?
    config = YAML.load(ERB.new(config_file.read).result)
    if config["#{Rails.env}_rocketjob]
      options = (config['options']||{}).symbolize_keys
      options[:logger] = SemanticLogger::DebugAsTraceLogger.new('Mongo:rocketjob')
      RocketJob::Config.mongo_connection = Mongo::MongoClient.from_uri(config['uri'], options)
    end
    # It is also possible to store the jobs themselves in a separate MongoDB database
    if config["#{Rails.env}_rocketjob_work]
      options = (config['options']||{}).symbolize_keys
      options[:logger] = SemanticLogger::DebugAsTraceLogger.new('Mongo:rocketjob_work')
      RocketJob::Config.mongo_work_connection = Mongo::MongoClient.from_uri(config['uri'], options)
    end
  else
    puts "\nmongo.yml config file not found: #{config_file}"
  end
end
```

For an example config file, `config/mongo.yml`, see [mongo.yml](https://github.com/rocketjob/rocketjob/blob/master/test/config/mongo.yml)

### Standalone Configuration

When running `rocketjob` in a standalone environment without Rails, the MongoDB
connections will need to be setup as follows:

```ruby
options = {
  pool_size:    50,
  pool_timeout: 5,
  logger:       SemanticLogger::DebugAsTraceLogger.new('Mongo:Work'),
}

# For example when using a replica-set for high availability
uri = 'mongodb://mongo1.site.com:27017,mongo2.site.com:27017/production_rocketjob'
RocketJob::Config.mongo_connection = Mongo::MongoClient.from_uri(uri, options)

# Use a separate database, or even server for `RocketJob::SlicedJob` slices
uri = 'mongodb://mongo1.site.com:27017,mongo2.site.com:27017/production_rocketjob_slices'
RocketJob::Config.mongo_work_connection = Mongo::MongoClient.from_uri(uri, options)
```

## [Next: Architecture ==>](architecture.html)
