---
layout: default
---

## Features

* Reliable
    * If a worker process crashes while processing a job, the job is never lost
* Ability to keep the job itself on completion, along with its result if any
    * Useful for audit or other management purposes
    * Frequent jobs can be destroyed on completion to reduce clutter or disk storage requirements
* User definable callbacks per Job class. For example, send an email, or perform other notifications
    * before_start
    * before_complete
    * before_fail
    * before_retry
    * before_pause
    * before_resume
    * before_abort
* Custom retry behavior
    * When a job fails, the logic for retrying it is usually very specific to that
      particular job.
        * For example, retry the job again in 10 minutes, or retry immediately
          for up to 3 times, etc...
    * The `after_fail` callback can be used to implement your own custom automated
      retry behavior.
* Error Handling
    * The error and complete backtrace is kept for every job that fails to aid problem
      determination.
* Cron replacement
    * [rocketjob][0] is a great place to store jobs that need to run periodically
    * Using the `run_at` feature when a job finishes it can create a new instance
      of itself to run at the future time interval.
    * See [DirmonJob](https://github.com/rocketjob/rocketjob/blob/master/lib/rocket_job/jobs/dirmon_job.rb) for a great example of a recurring job.

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
