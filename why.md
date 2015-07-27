---
layout: default
---

## Why rocketjob?

We have tried for years to make both [resque][1] and more recently [sidekiq][2]
work for large high performance batch processing.
Even [sidekiq-pro][3] was purchased and used in an attempt to process large batches.

Unfortunately, after all the pain and suffering with the existing asynchronous
worker solutions none of them have worked in our production environment without
significant hand-holding and constant support. Mysteriously the odd record/job
was disappearing when processing 100's of millions of jobs with no indication
where those lost jobs went.

In our environment we cannot lose even a single job or record, as all data is
business critical. The existing batch processing solutions do not supply any way
to collect the output from batch processing and as a result every job has custom
code to collect it's output.

High availability and high throughput were being limited by how much we could get
through [redis][4]. Being a single-threaded process it is constrained to a single
CPU. Putting [redis][4] on a large multi-core box does not help since it will not
use more than one CPU at a time.
Additionally, [redis][4] is constrained to the amount of physical memory is available
on the server.
[redis][4] worked very well when processing was below around 100,000 jobs a day.
When our workload increased to over 100,000,000 a day it could not keep
up. Its single CPU would often hit 100% CPU utilization when running many [sidekiq-pro][3]
servers. We also had to store actual job data in a separate MySQL database since
it would not fit in the available physical memory on the [redis][4] server.

[rocketjob][0] was created out of necessity due to constant support. End-users were
constantly contacting the development team to ask on the status of "hung" or
"in-complete" jobs, as part of our DevOps role.

Another significant production support challenge is trying to get [resque][1] or [sidekiq][2]
to process the batch jobs in a very specific order. Switching from queue-based
to priority-based job processing means that all jobs are processed in the order of
their priority and not what queues are defined on what servers and in what quantity.
This approach has allowed us to significantly increase the CPU and IO utilization
across all worker machines. The traditional queue based approach required constant
tweaking in the production environment to try and balance workload without overwhelming
any one server.

End-users are now able to modify the priority of their various jobs at runtime
so that they can get that business critical job out first, instead of having to
wait for other jobs of the same type/priority to finish first.

With `rocketjob-pro` it can upload the entire file, or all the data required for
processing the job and does not necessitate jobs having to store their data elsewhere.
Additionally, `rocketjob-pro` supports encryption and compression of any data uploaded
into Sliced Jobs to ensure PCI compliance and to prevent sensitive information from being exposed
either at rest in the data store, or in flight as it is being read or written to the
backend data store.
Often large files received for processing contain sensitive data that must not be exposed
in the backend job store. Having this capability built-in ensures all our jobs
are properly securing sensitive data.

Since moving to [rocketjob][0] our production support has diminished and now we can
focus on writing code again. :)

[0]: http://rocketjob.io
[1]: https://github.com/resque/resque
[2]: https://github.com/mperham/sidekiq
[3]: http://sidekiq.org/pro/
[4]: http://redis.io
