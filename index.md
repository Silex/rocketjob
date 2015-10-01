---
layout: default
---

[rocketjob][0] is the next generation background job processing system for [Ruby](http://ruby-lang.org), [JRuby](http://jruby.org) and [Rubinius](http://rubini.us).

Run jobs quickly and reliably in the background. Check on job status and even get its result when
the job is finished. Jobs written in Ruby are easily and reliably executed in the background across a cluster of workers.

<img src="images/rocket/rocket-icon-256x256.png" alt="rocketjob">

* Visibility
    * Every job visible in Web UI
* Reliable
* High performance
* High Throughput - Concurrent - fully utilize worker clusters

### Fire and forget job:

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

If the job fails it will be visible in [Mission Control][1] and be retried.

### Future dated jobs

To run the job in the future, use `run_at`:

```ruby
ImportJob.create!(
  file_name: 'file.csv',
  # Only run this job 2 hours from now
  run_at: 2.hours.from_now
)
```

### High priority jobs

When jobs are defined their default priority can be set. The priority of a
job is any value from 1 to 100, where 1 is the highest priority and 100 is the
lowest priority. The default priority for a job is 50.

Sometimes we want a specific instance of the job to have a higher priority than
others in the system:

```ruby
ImportJob.create!(
  file_name: 'file.csv',
  # Give this job a higher priority so that it will jump the queue
  priority: 5
)
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

  def perform
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

  key :count, Integer

  def perform
    # The output from this method is stored in the job itself
    { calculation: count * 1000 }
  end
end
```

Queue the job for processing:

```ruby
job = CalculateJob.create!(count: 24)

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
ImportJob.create!(
  file_name: 'file.csv',
  # Don't process this job if it is queued for longer than 15 minutes
  expires_at: 15.minutes.from_now
)
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

## Start a worker

To start a [rocketjob][0] worker process in which to run worker threads, run the following command
from your projects command line directory:

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

 * Ruby 1.9.3, 2.0, 2.1, 2.2, or greater
 * JRuby 1.7, 9.0.1.0, or greater
 * Rubinius 2.5, or greater

### Dependencies

[rocketjob][0] stores job data in the open source NOSQL data store [MongoDB][3].
[MongoDB][3] V2.6 or greater is required.

### [Next: Mission Control ==>][1]

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
