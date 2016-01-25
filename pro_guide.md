---
layout: default
---
## Rocket Job Pro Programmers Guide

The following features are available as part of Rocket Job Pro:

### Cron replacement

* [Rocket Job][0] is a great replacement for all those cron jobs.
* Include the `RocketJob::Plugins::Cron` plugin into any job to turn it into a cron job.
* Set the attribute `cron_schedule` to any valid cron schedule.
* `cron_schedule` also supports an optional timezone. If not set it defaults to the local timezone.

Example, run the job every night at midnight UTC:

```ruby
class MyCronJob < RocketJob::Job
  include RocketJob::Plugins::Batch
  include RocketJob::Plugins::Cron

  rocket_job do |job|
    # Every night at midnight UTC
    job.cron_schedule      = '0 0 * * * UTC'
  end

  def perform
    # Will be called every night at midnight UTC
  end
end
```

The `cron_schedule` will be validated when the job is saved, and is required for every job that
includes the `RocketJob::Plugins::Cron` plugin.

Benefits over regular cron:

* Easily run a Cron job immediately, via [Rocket Job Mission Control][1], or

```ruby
MyCronJob.first.update_attributes(run_at: nil)
```
* Easily change the cron schedule at any time.

```ruby
MyCronJob.first.update_attributes(cron_schedule: '* 1 * * * America/New_York')
```
* Cron job failures are viewable in [Rocket Job Mission Control][1]
* No single point of failure.
    * Linux/Unix cron is defined on a single server. If that server is unavailable for any reason
      then the cron jobs no longer run.
    * Rocket Job will run the cron jobs on any available worker.

### Batch / parallel processing

Regular jobs will run on a single worker. In order to scale up and use all available workers
its is necessary to break up the input data into "slices" so that different parts of the job
can be pocessed in parallel.

Jobs that include `RocketJob::Plugins::Batch` break their work up into slices so that many workers can work
on the individual slices at the same time. Slices take a large and unwieldy batch job and break it up
into "bite-size" pieces that can be processed a slice at a time by the workers.

Because Batch jobs consists of lots of smaller slices they can be paused, resumed, or even aborted as a whole.
If there are any failed slices when the job finishes, they can all be retried by retrying the job itself.

For example, using the default `slice_size` of 100, if the file contains 1,000,000
lines then this job will contain only 10,000 slices.

A running batch Job will be interrupted if a new job with a higher priority is queued for
processing.  This allows low priority jobs to use all available resources until a higher
priority job arrives, and then to resume processing once the higher priority job is complete.

```ruby
class ReverseJob < RocketJob::Job
  include RocketJob::Plugins::Batch

  rocket_job do |job|
    # Number of lines/records for each slice
    job.slice_size          = 100

    # Keep the job around after it has finished
    job.destroy_on_complete = false

    # Collect any output from the job
    job.collect_output      = true
  end

  def perform(line)
    # Work on a single record at a time across many workers
  end
end
```

Queue the job for processing:

```ruby
# Words would come from a database query, file, etc.
words = %w(these are some words that are to be processed at the same time on many workers)

job = ReverseJob.new

# Load words as individual records for processing into the job
job.upload do |records|
  words.each do |word|
    records << word
  end
end

# Queue the job for processing
job.save!
```

### Batch Output

Display the output from the above batch:

```ruby
# Display the results that were returned
job.output.each do |slice|
  slice.each do |record|
    # Display each result returned from job
    puts record
  end
end
```

The order of the output gathered above is exactly the same as the order in which the records
were uploaded into the job. This makes it easy to correlate an input record with its corresponding output.

### Batch Large File Processing

[Rocket Job Pro][2] supports very large files. It can easily upload
entire files into the Job for processing and automatically slices up the records in
the file into slices for processing.

Queue the job for processing:

```ruby
job = ReverseJob.new
# Upload a file into the job for processing
job.upload('myfile.txt')
job.save!
```

When complete, download the results of the batch into a file:

```ruby
# Download the output and compress the output into a GZip file
job.download('reversed.txt.gz')
```

[Rocket Job Pro][2] has built-in support for reading and writing

* `Zip` files
* `GZip` files
* files encrypted with [Symmetric Encryption][3]
* delimited files
    * Windows CR/LF text files
    * Linux text files
    * Auto-detects Windows or Linux line endings
    * Any custom delimiter
* files with fixed length records

Note:

* In order to read and write `Zip` on Ruby MRI, add the gem `rubyzip` to your `Gemfile`.
* Not required with JRuby since it will use the native `Zip` support built into Java

### Encryption

By setting the Batch Job attribute `encrypt` to true, input and output data is encrypted.
Encryption helps ensure sensitive data meets compliance requirements both at rest and in-flight.

```ruby
class ReverseJob < RocketJob::Job
  include RocketJob::Plugins::Batch

  rocket_job do |job|
    # Encrypt input and output data
    job.encrypt = true
  end

  def perform(line)
    line.reverse
  end
end
```

### Compression

Compression reduces network utilization and disk storage requirements.
Highly recommended when processing large files, or large amounts of data.

```ruby
class ReverseJob < RocketJob::Job
  include RocketJob::Plugins::Batch

  rocket_job do |job|
    # Compress input and output data
    job.compress = true
  end

  def perform(line)
    line.reverse
  end
end
```

### Worker Limiting / Throttling

[Rocket Job Pro][2] has the ability to throttle the number of workers that can work on
a batch job instance at any time.

Limiting can be used when too many concurrent workers are:

* Overwhelming a third party system by calling it too frequently.
* Impacting the online production systems by writing too much data too quickly to the master database.

Worker limiting also allows batch jobs to be processed concurrently instead of sequentially.

The `max_active_workers` throttle can be changed at any time, even while the job is running to
either increase or decrease the number of workers working on that job.

```ruby
class ReverseJob < RocketJob::Job
  include RocketJob::Plugins::Batch

  rocket_job do |job|
    # No more than 10 workers should work on this job at a time
    job.max_active_workers = 10
  end

  def perform(line)
    line.reverse
  end
end
```

### Directory Monitor

Directory Monitor can be used to monitor directories for new files and then to
load the entire file into the job for processing. The file is then either archived,
or deleted based on the configuration for that path.

### Multiple Output Files

[Rocket Job Pro][2] can also create multiple output files by categorizing the result
of the perform method.

This can be used to output one file with results from the job and another for
outputting for example the lines that were too short.

```ruby
class MultiFileJob < RocketJob::Job
  include RocketJob::Plugins::Batch

  rocket_job do |job|
    job.collect_output      = true
    job.destroy_on_complete = false
    # Register additional `:invalid` output category for this job
    job.output_categories   = [ :main, :invalid ]
  end

  def perform(line)
    if line.length < 10
      # The line is too short, send it to the invalid output collection
      Result.new(line, :invalid)
    else
      # Reverse the line ( default output goes to the :main output collection )
      line.reverse
    end
  end
end
```

When complete, download the results of the batch into 2 files:

```ruby
# Download the regular results
job.download('reversed.txt.gz')

# Download the invalid results to a separate file
job.download('invalid.txt.gz', category: :invalid)
```

## Error Handling

Since [Rocket Job Pro][2] breaks a single job into slices, individual records within
slices can fail while others are still being processed.

```ruby
# Display the exceptions for failed slices:
job = RocketJob::Job.find('55bbce6b498e76424fa103e8')
job.input.each_failed_record do |record, slice|
  p slice.exception
end
```

Once all slices have been processed and there are only failed slices left, then the job as a whole
is failed.

### [Next: Directory Monitor ==>](dirmon.html)

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: pro.html
[3]: http://reidmorrison.github.io/symmetric-encryption
