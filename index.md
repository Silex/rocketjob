---
layout: default
---

## Introduction

Run work in the background, check on its status and even get its result when
the job is finished.

<img src="images/rocket/rocket-icon-256x256.png" alt="rocketjob">

### Fire and forget job:

Many jobs are relatively simple, just go off and do some task.

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

If the job fails it will be visible in mission control and can be retried.

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

### Expired jobs

Sometimes queued requests for processing are no longer business relevant if not
completed by the worker by a specific date and time.

The system can queue a job for processing, but if the workers are too busy with
other higher priority jobs and are not able to process this job by its expiry
time, then discard the job without processing it at all:

```ruby
ImportJob.perform_later("file.csv") do |job|
  # Don't process this job if it is queued for longer than 15 minutes
  job.expires_at = 15.minutes.from_now
end
```

### Jobs with results

Run a job and when it is finished, get the result:

```ruby
class CalculateJob < RocketJob::Job
  rocketjob do |job|
    # Don't destroy the job when it completes
    job.destroy_on_complete = false
    # Collect the output from the perform method
    job.collect_output      = true
  end

  def perform(value)
    # Some long running calculation here, or multiple database calls etc.
    { calculation: value * 1000 }
  end
end
```

Queue the job for processing:

```ruby
job = CalculateJob.perform_later(24)

# Continue doing other work while the job runs
```

Since the job is just a document stored in [MongoDB][3] we can check on it's
status at any time:

```ruby
# Update the job's in memory state
job.reload

puts "Job's current state: #{job.state}"

puts "Full job status as shown in mission control: #{job.status.inspect}"

if job.completed?
  puts "Yay, it is finished"
  puts "The result of the calculation: #{job.result}"
end
```

### Start a worker

To start a worker to process the jobs above:

```ruby
bin/rocketjob
```

## Installation

Add the [rocketjob][0] gem to your Gemfile

```ruby
   gem 'rocketjob', '~> 0.10'
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

## Management

[rocketjob mission control][1] is the web interface for viewing and managing [rocketjob][0] jobs.
It is a Rails Engine that can be loaded into any Rails project.
[rocketjob mission control][1] can also be run stand-alone in a shell Rails application.

By separating [rocketjob mission control][1] into a separate gem means it does not
have to be loaded everywhere [rocketjob][0] jobs are defined or run.

## Installation

Add the [rocketjob mission control][1] gem to your Gemfile

```ruby
   gem 'rocketjob_mission_control', '~> 0.10'
```

Now run `bundle` to install [rocketjob mission control][1]

If not running `bundler`, just run:

```
   gem install rocketjob_mission_control
```

### Dependencies

[rocketjob][0] requires:

* MongoDB V2.6 or greater

### Compatibility

 * Ruby 1.9, 2.0, 2.1, 2.2, or greater
 * JRuby 1.7, 9.0.0.0, or greater
 * Rubinius 2.5, or greater

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
