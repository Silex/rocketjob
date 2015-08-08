---
layout: default
---

Run jobs quickly and reliably in the background. Check on job status and even get its result when
the job is finished.

<img src="images/rocket/rocket-icon-256x256.png" alt="rocketjob">

[rocketjob][0] is a high performance, reliable, concurrent, background job processing system for [Ruby](http://ruby-lang.org), [JRuby](http://jruby.org) and [Rubinius](http://rubini.us).
Jobs written in Ruby are easily and reliably executed in the background across a cluster of workers.

### Fire and forget job:

```ruby
class ImportJob < RocketJob::Job
  def perform(value)
    # Perform work here, such as processing a large file
  end
end
```

Queue the job for processing:

```ruby
ImportJob.perform_later("file.csv")
```

If the job fails it will be visible in [Mission Control][1] and be retried.

### Future dated jobs

To run the job in the future, use `run_at`:

```ruby
ImportJob.perform_later("file.csv") do |job|
  # Only run this job 2 hours from now
  job.run_at = 2.hours.from_now
end
```

### High priority jobs

When jobs are defined their default priority can be set. The priority of a
job is any value from 1 to 100, where 1 is the highest priority and 100 is the
lowest priority. The default priority for a job is 50.

Sometimes we want a specific instance of the job to have a higher priority than
others in the system:

```ruby
ImportJob.perform_later("file.csv") do |job|
  # Give this job a higher priority so that it will jump the queue
  job.priority = 5
end
```

### Job retention

Completed jobs can be retained so that they can be viewed in [Mission Control][1],
or retained for audit or support reasons. 

```ruby
class CalculateJob < RocketJob::Job
  rocketjob do |job|
    # Retain the job when it completes
    job.destroy_on_complete = false
  end

  def perform(value)
    # Perform work here 
  end
end
```


### Job output

Run a job and when it is finished, store the result:

```ruby
class CalculateJob < RocketJob::Job
  rocketjob do |job|
    # Don't destroy the job when it completes
    job.destroy_on_complete = false
    # Collect the output from the perform method
    job.collect_output      = true
  end

  def perform(value)
    # The output from this method is stored in the job itself 
    { calculation: value * 1000 }
  end
end
```

Queue the job for processing:

```ruby
job = CalculateJob.perform_later(24)

# Continue doing other work while the job runs

if job.reload.completed?
  puts "Job result: #{job.result}"
end
```

### Job status

Since the job is just a document stored in [MongoDB][3] we can check on it's
status at any time:

```ruby
# Update the job's in memory state
job.reload

# Current state ( For example: :queued, :running, :completed. etc. )
puts "Job is: #{job.state}"

# Complete state information as displayed in mission control
puts "Full job status: #{job.status.inspect}"
```

### Expired jobs

Sometimes queued requests for processing are no longer business relevant if not
completed by the worker by a specific date and time.

The system can queue a job for processing, but if the workers are too busy with
other higher priority jobs and are not able to process this job by its expiry
time, then the job will be discarded without processing:

```ruby
ImportJob.perform_later("file.csv") do |job|
  # Don't process this job if it is queued for longer than 15 minutes
  job.expires_at = 15.minutes.from_now
end
```

### Error Handling

The error and complete backtrace is kept for every job that fails to aid in
problem determination.

### Callbacks

User definable callbacks per Job class. 

For example, send an email, or perform other notifications

* before_start
* before_complete
* before_fail
* before_retry
* before_pause
* before_resume
* before_abort

### Custom retry behavior

When a job fails, the logic for retrying it is usually very specific to that
particular job.

* For example, retry the job again in 10 minutes, or retry immediately
  for up to 3 times, etc...
* The `after_fail` callback can be used to implement your own custom automated
  retry behavior.

### Cron replacement

* [rocketjob][0] is a great place to store jobs that need to run periodically
* Using the `run_at` feature when a job finishes it can create a new instance
  of itself to run at the future time interval.
* See [DirmonJob](https://github.com/rocketjob/rocketjob/blob/master/lib/rocket_job/jobs/dirmon_job.rb) for a great example of a recurring job.

### Reliability

If a worker process crashes while processing a job, the job remains in the queue and is never lost.
When the _worker_ instance is destroyed it's running jobs are re-queued amd will be processed
by another _worker_.

### Scalability

As workload increases greater throughput can be achieved by adding more servers. Each server
adds more CPU, Memory and local disk to process more jobs.

[rocketjob][0] scales linearly, meaning doubling the worker servers should double throughput. 
Bottlenecks tend to be databases, networks, or external suppliers that are called during job 
processing. Additional database slaves can be added to scale for example, MySQL, Postgres, 
along with sending database reads to the background job workers.

### High performance logging

Supports sending log messages, exceptions, and errors to one or more of:

* File
* Bugsnag
* MongoDB
* NewRelic
* Splunk
* Syslog (TCP, UDP, & local)
* Any user definable target via custom appenders

## Start a worker

To start a [rocketjob][0] worker process in which to run worker threads:

```ruby
bin/rocketjob
```

## Mission Control

[Mission Control][1] is the web interface for viewing and managing [rocketjob][0] jobs.
It is a Rails Engine that can be loaded into any Rails project.
[Mission Control][1] can also be run stand-alone in a shell Rails application.

By separating [Mission Control][1] into a separate gem means it does not
have to be loaded everywhere [rocketjob][0] jobs are defined or run.

## Installation

Add the [rocketjob][0] gem to your Gemfile

```ruby
   gem 'rocketjob'
```

Now run `bundle` to install [rocketjob][0].

If not running `bundler`, just run:

```
   gem install rocketjob
```

### MongoDB

[rocketjob][0] stores jobs in the open source data store [MongoDB][3].
Installing [MongoDB][3] is as easy as installing redis.

Installing [MongoDB][3] on a Mac running homebrew:

```
brew install mongodb
```

Then follow the on-screen instructions to start [MongoDB][3].

For other platforms, see [MongoDB Downloads](https://www.mongodb.org/downloads)

### Dependencies

[rocketjob][0] requires:

* MongoDB V2.6 or greater

### Compatibility

 * Ruby 1.9, 2.0, 2.1, 2.2, or greater
 * JRuby 1.7, 9.0.0.0, or greater
 * Rubinius 2.5, or greater

## [Next: Mission Control ==>][1]

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
