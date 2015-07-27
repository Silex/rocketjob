---
layout: default
---

## rocketjob-pro

The features below are only available through the add-on `rocketjob-pro` gem:

### Batch Processing

`rocketjob-pro` supports uploading the entire contents of file into a single job
for processing. The file is broken into slices of around 100 rows/records/lines
each. These "bite-size" slices are then processed concurrently by all the available
workers. Uploading a file is in essence a single job and with `rocketjob-pro` it
can now be managed as such.

For example, if a single file contains 1,000,000 lines it will result in 1 job
in [rocketjob][0] which consists of 10,000 slices, assuming we are using the default
`slice_size` of 100. This makes it very simple to process and monitor this single
job in [rocketjob mission control][1].

By converting the large file into a single job of "slices" it is easy to determine
how far the job is, or pause the job if needed, or retry all the failed slices if any.

Slices take a large and unwieldy batch job and break it up into "bite-size" pieces
that can be processed a slice at a time by the workers. Each slice is processed
in it's entirety by a worker before moving onto the next one. This makes it more
scalable since the system is not trashing trying to process the 1,000,000 lines
as separate jobs. Ideally each slice should take a worker between 30 seconds and
3 minutes to complete. The slice size is configurable by setting the `slice_size`
property on the job when it is defined.

A batch job does not only have to be one that is created from a file, it can hold
any data that is needed as input for the job.

Since each "batch" job is just one job with lots of slices, it is easy to pause
the job if needed, and then resume it later. Additionally it's priority can be
changed as needed.

Batch Jobs can also retain any output from processing the entire job so that
it can be downloaded into a file, or consumed for subsequent processing.

#### Throttling

`rocketjob-pro` has the ability to throttle the number of workers that can work on
a specific job instance at any time.

Sometimes there are jobs that we don't want to allow to use all available workers.
Too many concurrent workers could for example:

* Overwhelm a third party system
* Affect online production systems by writing too much data to the master database

Or, we may want to limit the number of workers working on the same job instance
so that several jobs at the same priority can be run at concurrently.

For example, we use the throttling mechanism to limit how many workers are
re-encrypting all the data in our database as part of our key-rotation for
PCI Compliance reasons. In this way the re-encryption can run constantly in
the background without impacting online traffic that is extremely latency sensitive.

The throttle can be changed at any time while the job is running to either increase
or decrease the number of workers working on that job. This ability allows us to
quickly change the number of workers depending on the impact on system and third
party resources.

### Large file support

RocketJob Pro has out of the box support for very large files. It can easily upload
entire files into the Job for processing. It automatically slices up the records in
the file, based on some delimeter or newline, and puts them into slices for processing.

### Directory Monitor

Directory Monitor can be used to monitor directories for new files and then to
load the entire file into the job for processing. The file is then either archived,
or deleted based on the configuration for that path.

### Encryption

PCI Compliance

### Compression

Reduce network utilization and disk storage requirements.

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
