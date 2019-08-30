# laravel-aws-worker
[![Build Status](https://travis-ci.org/dusterio/laravel-aws-worker.svg)](https://travis-ci.org/dusterio/laravel-aws-worker)
[![Code Climate](https://codeclimate.com/github/dusterio/laravel-aws-worker/badges/gpa.svg)](https://codeclimate.com/github/dusterio/laravel-aws-worker/badges)
[![Total Downloads](https://poser.pugx.org/dusterio/laravel-aws-worker/d/total.svg)](https://packagist.org/packages/dusterio/laravel-aws-worker)
[![Latest Stable Version](https://poser.pugx.org/dusterio/laravel-aws-worker/v/stable.svg)](https://packagist.org/packages/dusterio/laravel-aws-worker)
[![Latest Unstable Version](https://poser.pugx.org/dusterio/laravel-aws-worker/v/unstable.svg)](https://packagist.org/packages/dusterio/laravel-aws-worker)
[![License](https://poser.pugx.org/dusterio/laravel-aws-worker/license.svg)](https://packagist.org/packages/dusterio/laravel-plain-sqs)

Run Laravel (or Lumen) tasks and queue listeners inside of AWS Elastic Beanstalk workers

## Credit

Package created by:

![dusterio](https://github.com/dusterio/laravel-aws-worker)

## Installation

Create a web beanstalk environment and a worker beanstalk environment.

## Dependencies

* PHP >= 7.1.3
* Laravel >= 5.8

## Installation via Composer

To install simply run:

```
composer require charliepage88/laravel-aws-worker
```

### Worker

#### Configuring the queue

Every time you create a worker environment in AWS, you are forced to choose two SQS queues – either automatically generated ones or some of your existing queues. One of the queues will be for the jobs themselves, another one is for failed jobs – AWS calls this queue a dead letter queue.

You can set your worker queues either during the environment launch or anytime later in the settings:

![AWS Worker queue settings](https://www.mysenko.com/images/worker_settings.jpg)

Don't forget to set the HTTP path to ```/worker/queue``` – this is where AWS will hit our application. If you chose to generate queues automatically, you can see their details later in SQS section of the AWS console:

![AWS SQS details](https://www.mysenko.com/images/sqs_details.jpg)

You have to tell Laravel about this queue. First set your queue driver to SQS in ```.env``` file:

```
QUEUE_DRIVER=sqs
```

Then go to ```config/queue.php``` and copy/paste details from AWS console:

```php
        ...
        'sqs' => [
            'driver' => 'sqs',
            'key' => 'your-public-key',
            'secret' => 'your-secret-key',
            'prefix' => 'https://sqs.us-east-1.amazonaws.com/your-account-id',
            'queue' => 'your-queue-name',
            'region' => 'us-east-1',
        ],
        ...
```

To generate key and secret go to Identity and Access Management in the AWS console. It's better to create a separate user that ONLY has access to SQS.

### Notes

Create a `cron.yaml` file inside project root with following contents:

```yaml
version: 1
cron:
 - name: "schedule"
   url: "/worker/schedule"
   schedule: "* * * * *"
```

Inside of your `.env` file, set `REGISTER_WORKER_ROUTES` to `true`.

Update SQS config info to point to a different SQS queue than what the worker is using. During testing thus far, this has resulted in high accuracy of jobs being processed.

Do not have the scheduler run via cron, since the `cron.yaml` will take care of that.

Use supervisor to make sure the queue is running. When a job hits the worker, the queue will be necessary to enter the Job and process.

### Web

Inside of your `.env` file, set `REGISTER_WORKER_ROUTES` to `false`.

Update SQS config info to point to created worker environment.

Do not have the scheduler run via cron and make sure the queue is set to `sqs`.

Use supervisor to make sure the queue is running and sending jobs to the worker beanstalk.

## Errors and exceptions

Please make sure that two special routes are not mounted behind a CSRF middleware. Our POSTs are not real web forms and CSRF is not necessary here. If you have a global CSRF middleware, add these routes to exceptions, or otherwise apply CSRF to specific routes or route groups.

If your job fails, we will throw a ```FailedJobException```. If you want to customize error output – just customise your exception handler.
Note that your HTTP status code must be different from 200 in order for AWS to realize the job has failed.

## TODO

1. Add support for AWS dead letter queue (retry jobs from that queue?)

## Implications

Note that AWS cron doesn't promise 100% time accuracy. Since cron tasks share the same queue with other jobs, your scheduled tasks may be processed later than expected. 

## Post scriptum

I wrote a [blog post](https://blog.menara.com.au/2016/06/running-laravel-in-amazon-elastic-beanstalk/) explaining how this actually works.
