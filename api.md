---
layout: default
---

### Programmatic Interface

Aside from being able to see and change jobs through the [rocketjob mission control][1]
web interface it is often useful, and even highly desirable to be able to access
the job programmatically while it is running.

To find the last job that was submitted:

```ruby
job = RocketJob::Job.last
```

To find a specific job, based on it's id:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
```

To change it's priority:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
job.priority = 32
job.save!
```

Or, to skip the extra save step, update any attribute of the job directly:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
job.set(priority: 32)
```

How long has the last job in the queue been running for?

```ruby
job = RocketJob::Job.last
puts "The job has been running for: #{job.duration}"
```

How many `MyJob` jobs are currently being processed?

```ruby
count = RocketJob::MyJob.where(state: :running).count
```

Retry all failed jobs in the system:

```ruby
RocketJob::Job.where(state: :failed).each do |job|
  job.retry!
end
```

RocketJob::Job uses the same ActiveModel implementation that is used in ActiveRecord
and thereby exposes a very familiar API for anyone familiar with Rails.

Since everything about this job is held in this one document, all
details about the job are accessible programmatically.

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
