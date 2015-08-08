---
layout: default
---

## Why rocketjob?

We have tried for years to make both [resque][1] and more recently [sidekiq][2]
work for large high performance batch processing.
Even [sidekiq-pro][3] was purchased and used in an attempt to process large batches.

Unfortunately, after all the pain and suffering with the existing background
job solutions none of them have worked in our production environment without
significant hand-holding and constant support. Mysteriously the odd job
was disappearing when processing 100's of millions of jobs with no indication of
where those lost jobs went.

In our environment we cannot lose even a single job or record, as all data is
business critical. The existing batch processing solutions do not supply any way
to collect the output from batch processing and as a result every job has custom
code to collect it's output.

[rocketjob][0] is similar to [Delayed::Job][7], [resque][1], and [sidekiq][2] in that it runs
jobs in the background. [rocketjob][0] was created to scale well beyond all of the
previous technologies and was developed out of sheer need when the previous
solutions were not able meet business needs.

The fundamental difference is that [rocketjob][0] uses a single [priority based queue](https://en.wikipedia.org/wiki/Priority_queue).
All jobs are placed in this queue and the job with the highest priority is processed
first. Jobs with the same priority are processed on a first-in first-out (FIFO) basis.

This differs from the traditional approach of separate queues for jobs which
quickly becomes cumbersome when there are for example over a hundred different
types of jobs.

[resque][1] and [sidekiq][2] both use a queue based solution which requires removing the job
to be processed from the queue. [sidekiq-pro][3] attempts to address the issue by using the
redis resilient api's that add significant overhead to the processing of each job.

The problem with queueing based solutions is that one part of the job is put on
the queue to be processed, another part has to be separately managed elsewhere to
hold the status of the job so that it can be viewed, and then there is often a third
part that needs to be stored somewhere that contains the actual data to be processed.

Since [resque][1] and [sidekiq][2] both use [redis][4], which is an in-memory data-store, we have
to be very careful how much data is given to the job when it is queued so that
the memory limitations of the server running [redis][4] are not exceeded.

Performance is also an issue, since processing millions of jobs in [resque][1] and [sidekiq][2]
eventually bumps up against the single threaded limitation of [redis][4]. [redis][4] is limited
to a single core on any server, regardless of how many cores are available.
The leads to significant slow down as the single core approaches 100% utilization.

[redis][4] is also constrained by the amount of physical memory that is available
on the server. That means having to store actual job data in a separate MySQL
database since it would not fit in the available physical memory on the [redis][4] server.

[rocketjob][0] was created out of necessity due to the constant support. As part of our
DevOps role we often contacted by end-users for us to investigate "hung" or
"in-complete" jobs.

Another significant production support challenge is trying to get [resque][1] or [sidekiq][2]
to process the batch jobs in the order required by the business. Switching from queue-based
to priority-based job processing means that all jobs are processed in the order of
their priority and not what queues are defined on what servers and in what quantity.
This approach has allowed us to significantly increase the CPU and IO utilization
across all worker machines. The traditional queue based approach required constant
tweaking in the production environment to try and balance workload without overwhelming
any one server.

End-users are now able to modify the priority of their various jobs at runtime
so that they can get that business critical job out first, instead of having to
wait for other jobs of the same type/priority to finish first.

With [rocketjob-pro][10] an entire file can be uploaded into the job itself. Relevant data
can be held within the job, including both input and output data, rather than
forcing jobs to have to store their input or output data in yet another data store.
Additionally, [rocketjob-pro][10] supports encryption and compression of any data uploaded
into Sliced Jobs to ensure PCI compliance and to prevent sensitive information from being exposed
either at rest in the data store, or whilst being transmitted over the network.
Often large files that need to be processed contain sensitive data that must be
encrypted. Having this capability built-in ensures that all our jobs are properly securing sensitive data.

[SidekiqPro][3] adds support to group together multiple jobs into a single batch. Meaning
that loading 1,000,000 records from a file will result in 1,000,000 Sidekiq jobs, each
one is processed separately.

In [rocketjob-pro][10], the above 1,000,000 records result in only 1 job in [rocketjob][0] making it
very simple to process and monitor in [rocketjob mission control][1]. These 1,000,000
records are broken up into slices that are then separately processed by workers.
Since each slice contains by default 100 records each, only 10,000 items are queued
up for processing under this one job.

Since moving to [rocketjob][0] our production support has diminished and now we can
focus on writing code again. :)

[0]: http://rocketjob.io
[1]: https://github.com/resque/resque
[2]: https://github.com/mperham/sidekiq
[3]: http://sidekiq.org/pro/
[4]: http://redis.io
[7]: https://github.com/collectiveidea/delayed_job/
[10]: pro.html
