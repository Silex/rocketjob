---
layout: default
---

## Overview

[rocketjob][0] is a global [priority based queue](https://en.wikipedia.org/wiki/Priority_queue)
All jobs are placed in a single global queue and the job with the highest priority
is processed first. Jobs with the same priority are processed on a first-in
first-out (FIFO) basis.

This differs from the traditional approach of separate queues for jobs which
quickly becomes cumbersome when there are for example over a hundred different
types of jobs.

The global priority based queue ensures that the workers are utilized to their
capacity without requiring constant manual intervention.

[rocketjob][0] is designed to handle hundreds of millions of concurrent jobs
that are often encountered in high volume batch processing environments.
It is designed from the ground up to support large batch file processing.
For example a single file that contains millions of records to be processed
as quickly as possible without impacting other jobs with a higher priority.

## Management

[rocketjob mission control][1] is the web interface for viewing and managing `rocketjob` jobs.
It is a Rails Engine that can be loaded into any Rails project.
[rocketjob mission control][1] can also be run stand-alone in a shell Rails application.

By separating [rocketjob mission control][1] into a separate gem means it does not
have to be loaded everywhere [rocketjob][0] jobs are defined or run.

### Example

Example job to run in a separate worker process

```ruby
class MyJob < RocketJob::Job
  # Method to call asynchronously by the worker
  def perform(email_address, message)
    # For example send an email to the supplied address with the supplied message
    send_email(email_address, message)
  end
end
```

To queue the above job for processing:

```ruby
MyJob.perform_later('jack@blah.com', 'lets meet')
```

### Installation

    gem install rocketjob
    gem install rocketjob_mission_control

### Dependencies

[rocketjob][0] requires:

* MongoDB V2.6 or greater

### Compatibility

 * Ruby 1.9, 2.0, 2.1, 2.2, or greater
 * JRuby 1.7, 9.0.0.0, or greater
 * Rubinius 2.5, or greater


[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
[2]: http://reidmorrison.github.io/semantic_logger