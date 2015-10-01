---
layout: default
---

### Application Programming Interface

Aside from being able to see and change jobs through the [rocketjob mission control][1]
web interface it is often useful, and even highly desirable to be able to access
the job programmatically while it is running.

To find the last job that was submitted:

```ruby
job = RocketJob::Job.last
```

To find a specific job, based on its id:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
```

To change its priority:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
job.priority = 32
job.save!
```

Or, to skip the extra save step, update any attribute of the job directly:

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')
job.update_attributes(priority: 32)
```

How long has the last job in the queue been running for?

```ruby
job = RocketJob::Job.last
puts "The job has been running for: #{job.duration}"
```

How many `MyJob` jobs are currently being processed?

```ruby
count = MyJob.where(state: :running).count
```

Retry all failed jobs in the system:

```ruby
RocketJob::Job.where(state: :failed).each do |job|
  job.retry!
end
```

Is a job still running?

```ruby
job = RocketJob::Job.find('55aeaf03a26ec0c1bd00008d')

if job.completed?
  puts "Finished!"
elsif job.running?
  puts "The job is being processed by worker: #{job.worker_name}"
end
```

`RocketJob::Job` uses the same ActiveModel implementation that is used in ActiveRecord
and thereby exposes a very familiar API for anyone familiar with Rails.

Since everything about this job is held in this one document, all
details about the job are accessible programmatically.

### Validations

The usual [Rails validations](http://guides.rubyonrails.org/active_record_validations.html)
are available since they are exposed by ActiveModel.

Example of `presence` and `inclusion` validations:

```ruby
class Job
  include MongoMapper::Document
  key :priority, Integer

  validates :priority, inclusion: 1..100
end

class SimpleJob < Job
  key :login, String

  validates_presence_of :login
end
```

### Extensibility - Inheritance

Specialized jobs can be created using existing behavior from other jobs. This allows base job classes
to be created that other classes can then inherit from instead of directly from `RocketJob::Job`.

For example, create a base class called `FileJob` with behavior that is useful to other jobs:

```ruby
# Abstract class to add custom behavior
class FileJob < RocketJob::Job
  # Add a custom property that all file jobs can use
  key :file_name, String

  def before_perform
    # Custom file specific behavior, for example upload a file for processing
    puts file_name
  end
end
```

Create a job that includes the above specialized behavior by deriving it from `FileJob`:

```ruby
# MyJob now inherits the behavior in FileJob
class MyJob < FileJob
  def perform
    # process data
  end
end
```

This approach follows inheritance hierarchies allowing any job's behavior to be shared or
specialized in any way without affecting jobs that do not require or want that behavior.

Not only can `FileJob` modify the behavior of what is processed, it has full access to
all workflow related events, to take action when an event occurs. `FileJob` can also
change state transitions if required.

For example:

```ruby
# Abstract class with custom behavior
class FileJob < RocketJob::Job
  # Add a custom property that all file jobs can use
  key :file_name, String

  def before_perform
    # Custom file specific behavior, for example upload a file for processing
    # file_name is available in any method in this job:
    puts file_name
  end

  def before_fail
    # Send an email, or take other action when a job fails
  end
end
```

[rocketjob-pro][4] makes extensive use of the above extensibility features to create a powerful
batch processing solution built on top of `RocketJob::Job`.

### Extensibility - Mix-ins

Sometimes specific behavior can be created and then mixed into a job when needed.

For example create a mix-in that uses a validation to ensure that only one instance
of a job is running at a time:

```ruby
# encoding: UTF-8
require 'active_support/concern'

module RocketJob
  module Concerns
    # Prevent more than one instance of this job class from running at a time
    module Singleton
      extend ActiveSupport::Concern

      included do
        validates_each :state do |record, attr, value|
          if where(state: [:running, :queued], _id: {'$ne' => record.id}).exists?
            record.errors.add(attr, 'Another instance of this job is already queued or running')
          end
        end
      end

    end
  end
end
```

Now `include` the above mix-in into a job:

```ruby
class MyJob < RocketJob::Job
  # Create a singleton job so that only one instance is ever queued or running at a time
  include RocketJob::Concerns::Singleton

  def perform
    # process data
  end
end
```

Queue the job, supplying the `file_name` that was declared and used in `FileJob`:

```ruby
MyJob.create!(file_name: 'abc.csv')
```

Trying to queue the job a second time will result in:

```ruby
MyJob.create!(file_name: 'abc.csv')
# => MongoMapper::DocumentNotValid: Validation failed: State Another instance of this job is already queued or running
```

## Reference

* [API Reference](http://www.rubydoc.info/gems/rocketjob/)

### [Next: rocketjob-pro ==>](pro.html)

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
[4]: http://rocketjob.io/pro

