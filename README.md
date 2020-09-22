**If you're viewing this at https://github.com/collectiveidea/delayed_job_active_record,
you're reading the documentation for the master branch.
[View documentation for the latest release
(4.1.4).](https://github.com/collectiveidea/delayed_job_active_record/tree/v4.1.4)**

# DelayedJob ActiveRecord Backend

[![Gem Version](https://img.shields.io/gem/v/delayed_job_active_record.svg)](https://rubygems.org/gems/delayed_job_active_record)
[![Build Status](https://img.shields.io/travis/collectiveidea/delayed_job_active_record.svg)](https://travis-ci.org/collectiveidea/delayed_job_active_record)
[![Coverage Status](https://img.shields.io/coveralls/collectiveidea/delayed_job_active_record.svg)](https://coveralls.io/r/collectiveidea/delayed_job_active_record)

## Installation

Add the gem to your Gemfile:

    gem 'delayed_job_active_record'

Run `bundle install`.

If you're using Rails, run the generator to create the migration for the
delayed_job table.

    rails g delayed_job:active_record
    rake db:migrate

## Problems locking jobs

You can try using the legacy locking code. It is usually slower but works better for certain people.

    Delayed::Backend::ActiveRecord.configuration.reserve_sql_strategy = :default_sql

## Upgrading from 2.x to 3.0.0

If you're upgrading from Delayed Job 2.x, run the upgrade generator to create a
migration to add a column to your delayed_jobs table.

    rails g delayed_job:upgrade
    rake db:migrate

That's it. Use [delayed_job as normal](http://github.com/collectiveidea/delayed_job).

## Additional features in this fork
 Your comments are welcome

### "fair" cpu sharing for jobs of different users
We have different users, for which we do several time-intensive background-jobs, like scaling a large numbers of Images. The standard strategy is "first in, first out", what can be frustration for a user with just 10 Images, uploaded just after an other user with thousands. We didn't want to deal with the priority-field and increase it dynamically when setting up the job, as it makes setting up a job complex.
So we introduced the "balance_id", which can just be set to the user_id for example.
    
    delay(:balance_id=>current_user.id).scale_image(image)

Downside: The lookup for jobs to do is slower, but not significant in our case. (a view thousands jobs in the table)

### fuzzy queues
Additionally we let the workers choose the defined queus to work of by mysql-"LIKE '%queue%', so one can easily define a groub of ques for a worker (eg. '_import', which will do image_import and document_import)

    RAILS_ENV=production nice bundle exec script/delayed_job  --queues=%_import_%,%_scale_% -n 3 start
