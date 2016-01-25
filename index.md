---
layout: default
---
## Rocket Job

Rocket Job is a distributed, priority-based, background job, computation system for Ruby.
Rocket Job makes it easy to reliably process data using jobs written in Ruby.

Outgrown existing Ruby background job processing solutions?

Upgrade to Rocket Job.
Or, start small with Rocket Job and seamlessly scale up to meet future business demands.

### Rocket Job Key Differentiators:

* High Performance.
* Reliable.
    * Reliably process and track every job.
* Scalable.
    * Scales from a single worker to thousands of workers across hundreds of servers.
* Monitoring.
    * Web Interface, called Rocket Job Mission Control.
    * Monitor and manage every job in the system.
    * View current status.
* Priority based processing.
    * Process jobs in business priority order.
    * Change job priority to push through business critical jobs.
* Validations.
    * Validate job parameters when the job is created, instead of only when the job is being executed.
* Callbacks.
    * Extensive callback hooks to customize job processing and behavior.
* Ease of use.
    * Similar interface to ActiveRecord models.
* Kick off Jobs when files arrive.
    * [Rocket Job Directory Monitor][4] monitors directories for new files and then
      kicks off a job to process that file.
* Works with or without Rails.
* Free and open source.
    * Community support.

### Rocket Job Pro Key Differentiators:

* Designed to meet Enterprise Batch Processing requirements.
* Encryption.
    * Needed to meet compliance regulations.
* Compression.
    * Reduced storage and network requirements.
* Proven.
    * Rocket Job is used daily in production environments processing large files with millions of
      records, as well as large jobs that walk through very large databases.
* Cron replacement.
* Batch Jobs.
    * Over 50x faster than the competition.
    * Parallel processing of large data sets.
    * Pause / Resume running jobs.
    * Change the priority of jobs while they are running to push a job through earlier, when needed.
* Batch framework can also support:
    * Analytics
    * ETL
    * Map Reduce
    * etc.
* Large file streaming support.
    * CSV, Xlsx, Zip, GZip, etc.
* Commercial Support.

### Example:

```ruby
class ImportJob < RocketJob::Job
  # Create a property called `file_name` of type String
  key :file_name, String

  def perform
    # Perform work here, such as processing a large file
    puts "The file_name is #{file_name}"
  end
end
```

Queue the job for processing:

```ruby
ImportJob.create!(file_name: 'file.csv')
```

Monitor and manage the job via [Rocket Job Mission Control][1].

### Reliable

If a worker process crashes while processing a job, the job remains in the queue and is never lost.
When the _worker_ instance is destroyed / cleaned up its running jobs are re-queued and will be processed
by another _worker_.

### Scalable

As workload increases greater throughput can be achieved by adding more servers. Each server
adds more CPU, Memory and local disk to process more jobs.

[Rocket Job][0] scales linearly, meaning doubling the worker servers should double throughput.
Bottlenecks tend to be databases, networks, or external suppliers that are called during job
processing.

Additional database slaves can be added to scale for example, MySQL, and/or Postgres.
Then configuring the job workers to read from the slaves helps distribute the load. For example, we use
[ActiveRecord Slave](https://github.com/reidmorrison/active_record_slave) to efficiently redirect
ActiveRecord MySQL reads to multiple slave servers.

### High performance logging

Supports sending log messages, exceptions, and errors simultaneously to one or more of:

* File
* Bugsnag
* MongoDB
* NewRelic
* Splunk
* Syslog (TCP, UDP, & local)
* Any user definable target via custom appenders

To remove the usual impact of logging, the log writing is performed in a separate thread.
In this way the time it takes to write to one or logging destinations does _not_ slow down
active worker threads.

## Start a Worker

Start a [Rocket Job][0] worker process:

```ruby
rocketjob
```

Or, if using Bundler:

```ruby
bundle exec rocketjob
```

Or, if you are using Bundler binstubs:

```ruby
bin/rocketjob
```

### Compatibility

* Ruby 1.9.3, 2.0, 2.1, 2.2, 2.3, or greater
* JRuby 1.7, 9.0.4.0, or greater
* [MongoDB][3] V2.6 or greater is required.

### [Next: Mission Control ==>][1]

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
[4]: dirmon.html
