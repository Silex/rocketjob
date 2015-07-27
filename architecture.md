---
layout: default
---

## Architecture

[rocketjob][0] is similar to delayedjob, resque, and sidekiq in that it runs
jobs in the background. [rocketjob][0] was created to scale well beyond all of the
previous technologies and was developed out of sheer need when the previous
solutions were not able meet business needs.

The fundamental difference is that [rocketjob][0] uses a single [priority based queue](https://en.wikipedia.org/wiki/Priority_queue).
All jobs are placed in this queue and the job with the highest priority is processed
first. Jobs with the same priority are processed on a first-in first-out (FIFO) basis.

This differs from the traditional approach of separate queues for jobs which
quickly becomes cumbersome when there are for example over a hundred different
types of jobs.

The priority based queue ensures that the workers are utilized to capacity
without requiring any manual intervention or tuning of workers.

[rocketjob][0] is designed to handle millions of concurrent jobs that are often
encountered in high volume batch processing environments.
It is designed from the ground up to support large batch file processing.
For example a single file that contains millions of records to be processed
as quickly as possible without impacting other jobs with a higher priority.

Like sidekiq, [rocketjob][0] also uses threads instead of processes to run each
job. This approach is much more scalable since it is more efficient by sharing
memory that would otherwise be duplicated if processes were used.

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
