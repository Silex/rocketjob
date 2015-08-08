---
layout: default
---

## All new architecture

RocketJob uses [MongoDB][6] to do "in-place" processing of a job. A job is only created
once and stored entirely as a single document in [MongoDB][6]. [MongoDB][6] is highly concurrent,
allowing all CPU's to be used if needed to scale out workers. [MongoDB][6] is not
only memory resident for performance, it can also write older data to disk, or
when there is not enough physical memory to hold all of the data.

This means that all information relating to a job is held in the one document:

* State: queued, running, failed, aborted, or completed
* Percent Complete
* Job arguments
* etc..

The status of any job is immediately visible in the [rocketjob mission control][1] web
interface, without having to update some other data store since the job only lives
in one place.

The single document approach for the job is possible due to a very efficient
modify-in-place feature in [MongoDB][6] called [`find_and_modify`](http://docs.mongodb.org/manual/reference/command/findAndModify/)
that allows jobs to be efficiently assigned to any one of hundreds of available
workers without the locking issues that befall relational databases.

### Priority based processing

Jobs are processed based on their priority which is specified when the job is defined,
and can be overridden on a per job instance basis. In fact the job's priority can be
changed at runtime by operations to make it jump the queue, or put it further down
the priority to allow other more important jobs to finish first.

The priority based queue ensures that the workers are utilized to capacity
without requiring any manual intervention or tuning of workers.

### Scalability

[rocketjob][0] is designed to handle millions of concurrent jobs that are often
encountered in high volume batch processing environments.
It is designed from the ground up to support large batch file processing.
For example a single file that contains millions of records to be processed
as quickly as possible without impacting other jobs with a higher priority.

### Concurrency

Like [sidekiq][3], [rocketjob][0] also uses threads instead of processes to run each
job. This approach is much more scalable since it is more efficient by sharing
memory that would otherwise be duplicated if processes were used.

### Extensibility

Since every job inherits from `RocketJob::Job` it can override any behavior that
the basic job has implemented:

* Callbacks are available for every state transition.
  * Custom code can be called before and after every transition.
  * Useful for firing emails or notifications as the job progresses or fails.
* Custom retry logic specific to a class of job can be implemented.
* The way the job is processed by the workers can be completely customized.

## [Next: Why? ==>](why.html)

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
[2]: https://github.com/resque/resque
[3]: https://github.com/mperham/sidekiq
[4]: http://sidekiq.org/pro/
[5]: http://redis.io
[6]: http://mongodb.org/
[7]: https://github.com/collectiveidea/delayed_job/
