---
layout: default
---

## rocketjob-pro

`rocketjob-pro` adds several enterprise features:

* Enterprise scale batch job processing
* Encryption
* Compression
* Direct support for large file processing
    * Supports
        * Zip
        * GZip
        * Files encrypted with Symmetric Encryption
        * Delimited and fixed length files
* Concurrency
    * A running sliced Job is interrupted so that a new job with a higher priority
      can be processed first.
    * This allows low priority jobs to use all available resources, until a higher
      priority job arrives, and then to resume processing once the higher priority
      job is complete, or when it no longer requires all available workers.
* Throttling
    * Throttle how many workers should work on a specific sliced job at a time.

The features below are only available with `rocketjob-pro`:

### Batch Processing

`rocketjob-pro` has out of the box support for very large files. It can easily upload
entire files into the Job for processing. It automatically slices up the records in
the file, based on some delimeter or newline, and puts them into slices for processing.

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

Slices take a large and unwieldy batch job and break it up into "bite-size" pieces
that can be processed a slice at a time by the workers.
The job can be paused, resumed, or even aborted as a whole. If there are any failed
slices when the job finishes, they can all be retried by hitting retry on the job itself.

For example, using the default `slice_size` of 100, if the file contains 1,000,000
lines then this job will contain only 10,000 slices.

Processing the output of a job instead of downloading it into a file:

```ruby
# Display the results that were returned
job.output.each do |slice|
  slice.each do |line|
    puts line
  end
end
```

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

The data uploaded into a sliced job can be encrypted while being uploaded
to secure sensitive data. Encryption ensure it meets for example PCI Compliance
requirements for sensitive data at rest and in-flight.

Similarly the output data can be encrypted in an sliced job to keep that sensitive
data private.

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

#### Throttling

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

`rocketjob-pro` has the ability to throttle the number of workers that can work on
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

### Directory Monitor

Directory Monitor can be used to monitor directories for new files and then to
load the entire file into the job for processing. The file is then either archived,
or deleted based on the configuration for that path.

### Multiple Output Files

`rocketjob-pro` can also create multiple output files by categorizing the result
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

Since `rocketjob-pro` breaks a single job into slices, individual records within
slices can fail while others are still being processed.

```ruby
# Display the exceptions for failed slices:
job = RocketJob::Job.find('55bbce6b498e76424fa103e8')
job.input.each_failed_record do |record, slice|
  p slice.exception
end
```

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
