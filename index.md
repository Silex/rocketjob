---
layout: default
---
## Rocket Job

Rocket Job, a distributed, priority based, background job, batch processing system for Ruby and Rails.
Rocket Job makes it easy to reliably process data using jobs written in Ruby.

### Rocket Job Key Differentiators:

* Monitoring.
    * Web Interface - Rocket Job Mission Control.
    * Monitor and manage every job in the system.
    * View current status.
* Priority based processing.
    * Process jobs in business priority order.
    * Dynamically change job priority to push through business critical jobs.
* Cron replacement.
* Validations.
    * Validate job parameters when the job is created.
* Callbacks / Middleware.
    * Extensive callback hooks to customize job processing and behavior.
* Ease of use.
    * Similar interface to ActiveRecord models.
* Trigger Jobs when new files arrive.
    * [Rocket Job Directory Monitor][4] monitors directories for new files and then
      kicks off a job to process that file.
* High Performance.
    * Over [1,000](rj_performance.html) jobs per second on a single server.
* Reliable.
    * Reliably process and track every job.
* Scalable.
    * Scales from a single worker to thousands of workers across hundreds of servers.
* Works with or without Rails.
* Free
    * Open Source.
    * Community support.

### Rocket Job Pro Key Differentiators:

Designed to meet Enterprise Batch Processing requirements.

* High Performance.
    * Process large batches at up to [500,000](rj_pro_performance.html) records per second using a single server.
    * Over 80x faster at batch processing than its nearest competitor.
* Encryption.
    * Meet compliance regulations.
* Compression.
    * Reduced storage and network requirements.
* Proven.
    * Rocket Job Pro is used daily in production environments processing large files with millions of
      records, as well as large jobs that walk through very large databases.
* Batch Jobs.
    * Parallel processing of large data sets.
    * Pause / Resume running jobs.
    * Change the priority of jobs while they are running to push a job through earlier, when needed.
* Batch framework also supports:
    * Analytics
    * ETL
    * Map Reduce
    * etc.
* Large file streaming support.
    * CSV, XlSX, Zip, GZip, etc.
* Commercial Support.

### Example:

~~~ruby
class ImportJob < RocketJob::Job
  # Create a property called `file_name` of type String
  key :file_name, String

  def perform
    # Perform work here, such as processing a large file
    puts "The file_name is #{file_name}"
  end
end
~~~

Queue the job for processing:

~~~ruby
ImportJob.create!(file_name: 'file.csv')
~~~

Monitor and manage any job via [Rocket Job Mission Control][1].

![Screen shot](images/rjmc_running.png)

### [Next: Compare ==>](compare.html)

[0]: http://rocketjob.io
[1]: mission_control.html
[2]: http://reidmorrison.github.io/semantic_logger
[3]: http://mongodb.org
[4]: dirmon.html
