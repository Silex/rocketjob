---
layout: default
---
## Rocket Job Pro Batch Performance Test

Setup for the performance test below:

* 13" MacBook Pro (Dual core i7, Early 2015)
* Mac OSX v10.11
* Ruby v2.3 (MRI/CRuby)
* MongoDB v3.2.1

### Start worker processes:

Run the following command in 3 separate windows:

```
bundle exec rocketjob --log_level warn --threads 5
```

### Performance Test

After starting the Rocket Job workers above, open an `irb` console, or Rails console,
and run the following code:

```ruby
job = RocketJob::Jobs::PerformanceJob.new(log_level: :warn, slice_size: 1000, compress: true, encrypt: true)
job.upload do |writer|
  10_000_000.times { |i| writer << i }
end
job.save!

puts 'Waiting for job to complete'
while (!job.reload.completed?)
  sleep 3
end

puts "Processing rate: #{(job.record_count / (job.completed_at - job.started_at)).round(3)} records/second"
```

The above job run results in numbers over 370,000 records per second.

By increasing the `slice_size` further, results in even higher processing rates.

Enabling or disabling compression and/or encryption does not appear to have a significant impact on processing times.
