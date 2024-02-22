# Working a Job

Dispatched jobs will get picked up by Worker Pools set up for the same connection and queue.  They are picked up in order by created date (FIFO).

If you have a valid Worker Pool, then you don't need to do anything else to have your job picked up.  If you want to see all the details of cbq dispatching and marshalling jobs, enable `debug` logging in LogBox for `cbq`.

When a Job is worked, the Job instance is created via WireBox and the memento hydrated into the Job component.  Then the `handle` method is called.

When a Job is completed, the Job will be removed from the persistent storage of the Queue Provider.  If the Job attempt fails, it will be sent back to the Queue Provider to be retried, up to the `maxAttempts` amount.  Once a Job has failed the `maxAttempts` amount it will be removed and marked as failed.  If you have `logFailedJobs` enabled, it will be sent to the [failed jobs table](failed-jobs.md#logging-failed-jobs).

## Inside \`handle\`

### release

You can choose to manually release a Job back to a queue with an optional delay (in seconds) using the `release` function.  This is useful when you need to delay processing for a Job, perhaps due to rate limiting or licensing constraints.

{% hint style="info" %}
Calling `release` does not stop processing the `handle` method, so make sure you `return` if you don't want to keep executing your `handle` method.
{% endhint %}

```cfscript
component {

    function handle() {
        this.release( 60 ); // in seconds
        return;
    }
    
}
```

## Interception Points

cbq fires a number of interception points during a Job's lifecycle.

### onCBQJobMarshalled

This is fired before the `handle` method is called on a Job.  The data includes the `job` to be executed.

```cfscript
variables.interceptorService.announce( "onCBQJobMarshalled", {
    "job" : job
} );
```

### onCBQJobComplete

This is fired after a Job completes successfully.  The data includes the `job` to be executed as potentially a `result` returned from the `Job`'s handle method.

```cfscript
variables.interceptorService.announce( "onCBQJobComplete", {
    "job" : job,
    "result" : isNull( result ) ? javacast( "null", "" ) : result
} );
```

## Tips

### Need up to date data?

Could your data change between when you dispatched a job and when you work the job?  If so, consider using the key as a property and fetching the data from inside the job.

### Need to use historical data?

In some cases, you want to work with the data that existed at the time the Job was dispatched.  In these cases, make sure to include all the needed data in the Job instance.

### Check if you need to work the Job still

Things might have changed since the Job was dispatched. Most Queue Providers do not support querying the current Jobs queue or interacting with it in advance, so check in your `dispatch` method if the Job still needs to be worked.

### Need to try the Job later?

In some cases, you need to retry your Job later.  In these cases, use the `release` method to send the Job back to the queue.  It also takes an optional `delay` which sets the `backoff` time for the next Job attempt.

You can combine this with setting the `maxAttempts` of a Job to `0` to retry it indefinitely.  Remember that a Job is only reattempted if it fails.  If you decide that a Job is finished, just `return`.

### Jobs can dispatch other Jobs

You can dispatch other Jobs from your Job.  This can be different Jobs or even a similar instance of the same Job.

## Why isn't my Job getting picked up?

To work a cbq job, you need a [defined Worker Pool in your cbq Config file.](../configuration/config-file/#newworkerpool) In order to work a Job, the Worker Pool must:

1. Be working the same [connection](../configuration/config-file/worker-pool.md#forconnection) as the Job.
2. Be working the same [queue](../configuration/config-file/worker-pool.md#onqueues) as the Job.
3. Have at least one active worker (defined with [`setQuantity`](../configuration/config-file/worker-pool.md#setquantity)).

If these requirements are not met, the job will stay in the queue storage, unable to get picked up.

