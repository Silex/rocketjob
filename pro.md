---
layout: default
---

## rocketjob-pro

[rocketjob-pro][2] adds several enterprise features

### Batch processing

Jobs can be broken up into slices so that a _single_ job can be processed concurrently by all
available workers

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # Number of lines/records for each slice
    job.slice_size = 100
  end

  def perform(line)
    # Work on a single record at a time across many workers
  end
end
```

Queue the job for processing:

```ruby
job = ReverseJob.perform_later do |job|
  # Words would come from a database query, file, etc.
  words = %w(these are some words that are to be processed at the same time on many workers)

  # Load words as individual records for processing into the job
  job.upload do |records|
    words.each do |word|
      record << word
    end
  end
end
```

### Batch processing and collect results

Collect the output from running a batch

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # Number of lines/records for each slice
    job.slice_size = 100

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
job = ReverseJob.perform_later do |job|
  # Words would come from a database query, file, etc.
  words = %w(these are some words that are to be processed at the same time on many workers)

  # Load words as records for processing into the job
  job.upload do |records|
    words.each do |word|
      record << word
    end
  end
end
```

Processing the output from the job:

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

### Large file processing

[rocketjob-pro][2] has out of the box support for very large files. It can easily upload
entire files into the Job for processing. It automatically slices up the records in
the file, and puts them into slices for processing.

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # Number of lines/records for each slice
    job.slice_size          = 100

    # Keep the job around after it has finished
    job.destroy_on_complete = false

    # Collect any output from the job
    job.collect_output      = true
  end

  def perform(line)
    # Process each line of the file
    line.reverse
  end
end
```

Queue the job for processing:

```ruby
job = ReverseJob.perform_later do |job|
  # Upload a file into the job for processing
  job.upload('myfile.txt')
end
```

When complete, download the results of the batch into a file:

```ruby
# Download the output and compress the output into a GZip file
job.download('reversed.txt.gz')
```

[rocketjob-pro][2] has built-in support for reading and writing

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

* In order to read and write `Zip` on Ruby MRI and Rubinius, add the gem `rubyzip` to your bundle.
* Not required with JRuby since it will use the native `Zip` support built into Java

### Encryption

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # Encrypt input and output data
    job.encrypt = true
  end

  def perform(line)
    line.reverse
  end
end
```

By setting `job.encrypt = true` the uploaded data is encrypted during the upload
to secure sensitive data. Encryption ensures it meets for example PCI Compliance
requirements for sensitive data at rest and in-flight.

Similarly, the output data will also be encrypted to ensure it remains secure.

### Compression

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # Compress input and output data
    job.compress = true
  end

  def perform(line)
    line.reverse
  end
end
```

Support for compression reduces network utilization and disk storage
requirements. Works extremely well when processing large text files.

### Rate limiting / Throttling workers

```ruby
class ReverseJob < RocketJob::SlicedJob
  rocketjob do |job|
    # No more than 10 workers should work on this job at a time
    job.max_active_workers = 10
  end

  def perform(line)
    line.reverse
  end
end
```

[rocketjob-pro][2] has the ability to throttle the number of workers that can work on
a specific sliced job instance at any time.

Sometimes there are jobs that we don't want to allow to use all available workers.
Too many concurrent workers could for example:

* Overwhelm a third party system.
* Affect the online production systems by writing too much data to the master database.

Or, we may want to limit the number of workers working on the same job instance
so that several jobs of the same priority can be run concurrently.

For example, we use the throttling mechanism to limit how many workers are
re-encrypting all the data in our database as part of our key-rotation for
PCI Compliance reasons. In this way the re-encryption can run constantly in
the background without impacting online traffic, which is extremely latency sensitive.

The throttle can be changed at any time while the job is running to either increase
or decrease the number of workers working on that job. This ability allows us to
quickly change the number of workers depending on the impact on system and third
party resources.

### Concurrency

Jobs derived from `RocketJob::SlicedJob` break their work up into slices so that many workers can work
on the individual slices at the same time.

Slices take a large and unwieldy batch job and break it up into "bite-size" pieces
that can be processed a slice at a time by the workers.

Because the job consists of lots of smaller slices it can be paused, resumed, or even aborted as a whole.
If there are any failed slices when the job finishes, they can all be retried by retrying the job itself.

For example, using the default `slice_size` of 100, if the file contains 1,000,000
lines then this job will contain only 10,000 slices.

A running sliced Job will be interrupted if a new job with a higher priority is queued for
processing.  This allows low priority jobs to use all available resources, until a higher
priority job arrives, and then to resume processing once the higher priority
job is complete.

### Directory Monitor

Directory Monitor can be used to monitor directories for new files and then to
load the entire file into the job for processing. The file is then either archived,
or deleted based on the configuration for that path.

### Multiple Output Files

[rocketjob-pro][2] can also create multiple output files by categorizing the result
of the perform method.

This can be used to output one file with results from the job and another for
outputting for example the lines that were too short.

```ruby
class MultiFileJob < RocketJob::SlicedJob
  rocket_job do |job|
    job.collect_output      = true
    job.destroy_on_complete = false
    # Register additional output categories for the job
    job.output_categories   = [ :invalid ]
  end

  def perform(line)
    if line.length < 10
      # The line is too short, send it to the invalid output collection
      Result.new(line, :invalid)
    else
      # Reverse the line
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
job(:invalid).download('invalid.txt.gz')
```

## Error Handling

Since [rocketjob-pro][2] breaks a single job into slices, individual records within
slices can fail while others are still being processed.

```ruby
# Display the exceptions for failed slices:
job = RocketJob::Job.find('55bbce6b498e76424fa103e8')
job.input.each_failed_record do |record, slice|
  p slice.exception
end
```

## [Next: Compare ==>](compare.html)

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: pro.html
[3]: http://reidmorrison.github.io/symmetric-encryption
