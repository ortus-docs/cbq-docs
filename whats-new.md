# What's New?

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
