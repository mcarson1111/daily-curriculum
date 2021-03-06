# Background jobs w/ Resque

Resque is a ruby library for creating background jobs, which are tasks that are placed in a queue and processed by a separate process than our running application. [Resque](https://github.com/resque/resque) is build upon [Redis](http://redis.io), redis is a key-value store, or referred to as a NoSQL database, the idea is basically that it's a database that stores hashes.

#### Install Redis

Resque depends on a redis server to be running.

```
brew update
brew install redis
```

To start the server, run:
```
redis-server
```

See [Try Redis](http://try.redis.io) for more info.


#### Resque

Add the resque gem to the `Gemfile`

```ruby
gem "resque"
```

Next we'll need to start an addition process to run in the background of our application. Resque provides a rake task to start this process, but first we need to include the tasks within our application, add the following to the `Rakefile`

```ruby
require "resque/tasks"
task "resque:setup" => :environment
```

Then to start the process you can use

```bash
bundle exec rake resque:work QUEUE='*'
```

Resque uses the idea of `Jobs` to create items for the a queue. We'll start by creating a new class in a jobs dir.

```bash
mkdir app/jobs
touch app/jobs/email_job.rb
```
In the `email_job.rb` create a new class:
```ruby
class EmailJob
end
```
Resque depends on two things, a `@queue` instance variable, and a class method called perform:

```ruby
class EmailJob
  @queue = :email
  def self.perform
    puts "THIS IS A JOB"
  end
end
```
We can then use the `Resque` gem to add something to this queue, in the rails console

```ruby
Resque.enqueue(EmailJob)
```

Mount the resque server to the rails app
```ruby
require 'resque/server'
Golf::Application.routes.draw do
  mount Resque::Server, :at => "/resque"
  ...
```

Now we can replace the contents of the `perform` method to actually do logic, we can also accepts arguments on the perform method:

```ruby
class EmailJob
  @queue = :email
  def self.perform(club_id)
    ClubMailer.join_us(club_id).deliver
  end
end
```

Be sure to restart the Resque rake task after changing any of the job classes. Now to enqueue this job, we will need to pass in an additional argument, which is the clubs id
```ruby
Resque.enqueue(EmailJob, Club.last.id)
```
Now our mailer can be sent in a background job using Resque
