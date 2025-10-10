# What's New?

## v4.0.0

### Breaking Change

On Batches, the `finally` and `catch` methods have been deprecated in order to support Adobe ColdFusion.

`finally` -> `onComplete`

`catch` -> `onFailure`

If you are running on Lucee or BoxLang, the old methods names will still work, but they may be removed in a future version.  We recommend migrating to the new method names.

## v3.0.2

* Add error logging around logging failed jobs.

## v3.0.1

* **DBProvider:** Fix releasing job timeouts using the wrong value

## v3.0.0

* The `failedDate` column now uses a Unix timestamp as the column type. This avoids any timezone issues and aligns more closely with the `cbq_jobs` table used by the `DBProvider`.
* Allow worker pools to finish currently running jobs, up to a configurable timeout.
* Add optional clean-up tasks for completed or failed jobs, failed job logs, and completed or cancelled batches.
* **DBProvider:** Improve database locking to avoid duplicate runs of the same job.
* Fixes unwrapping an optional in a log message.

## v2.1.0

Add back ability to [work on multiple queues](configuration/config-file/worker-pool.md#onqueue) on a per-Provider basis. Currently only the `DBProvider` supports it.

Add support for `before` and `after` [lifecycle methods](jobs/defining-a-job.md#lifecycle-methods) on a Job instance.

Add ability to [restrict interceptor execution ](interceptors.md#jobpattern-annotation)with a `jobPattern` annotation.  (This is similar to the `eventPattern` annotation [provided by ColdBox](https://coldbox.ortusbooks.com/the-basics/interceptors/restricting-execution).)

## v2.0.5

**DBProvider**: Disable `forceRun` because it is causing ColdBox Futures to lose mappings.

## v**2.0.4**

Reload module mappings in an attempt to work around ColdBox Async losing them.

## **v2.0.3**

**SyncProvider:** Add pool to releaseJob call

## v2.0.2

Fix moduleSettings missing a queryOptions key for failed jobs

## v2.0.1

ColdBoxAsyncProvider now correctly respects Worker Pool conifguration, including queues.

## v2.0.0

### BREAKING CHANGES

#### Worker Pools can only define a single queue to work

In order to work with new Queue Providers, the Worker Pools need to be updated to only work a specific queue. This is because many future Queue Providers like RabbitMQ and Amazon SQS only support listening to a single queue in a consumer.

If you previously had multiple queues defined in a Worker Pool, you will need to define multiple Worker Pool instances, one for each of the queues.

```cfscript
// Old
newWorkerPool( "default" )
    .forConnection( "default" )
    .onQueues( [ "priority", "default" ] );
    
// New
newWorkerPool( "default" )
    .forConnection( "default" )
    .onQueue( "priority" );
    
newWorkerPool( "default" )
    .forConnection( "default" )
    .onQueue( "default" );
```

Notice that the method has been renamed from `onQueues` to `onQueue`.

Additionally, there are no more wildcard queues. Every queue you publish to must have a WorkerPool defined in order for that Job to be worked.

Finally, queue priorities are defined by the number of workers (`quantity`) you define for the WorkerPool. WorkerPools can no longer share workers across queues.
