# Dispatching a Job

## Dispatching

Creating a Job is all well and good, but the Job will not be executed until it is dispatched on to a [Queue Connection](../configuration/config-file/queue-connection.md).

If you have a Job instance, you can dispatch it by calling the `dispatch` method.

```cfscript
component {

    function index( event, rc, prc ) {
        getInstance( "SendWelcomeEmailJob" )
            .setEmail( "john@example.com" )
            .setGreeting( "Welcome!" )
            .dispatch();
    }

}
```

You can also create and dispatch a job in the same call using the `cbq.dispatch()` method.

```cfscript
component {

    property name="cbq" inject="cbq@cbq";
    
    function index( event, rc, prc ) {
        cbq.dispatch(
            job = "SendWelcomeEmailJob",
            properties = {
                "email": "john@example.com",
                "greeting": "Welcome!"
            },
            chain = [],
            queue = "priority",
            connection = "db",
            backoff = 10,
            timeout = 30,
            maxAttempts = 3
        );
    }

}
```

## Interception Points

### onCBQJobAdded

This is fired before serializing a Job and sending it to a connection.  The data includes the `job` to be serialized and dispatched and the connection the Job is being dispatched on.

```cfscript
variables.interceptorService.announce( "onCBQJobAdded", {
    "job" : job,
    "connection" : connection
} );
```

## Observing publication (7.0.1+)

`onCBQJobAdded` precedes the provider call and does not establish that the job was published. The `onCBQJobPublished` and `onCBQJobPublishException` hooks observe the provider result for single and bulk dispatch. See [Queue Observability](../observability.md#publishing) for their payloads, failure behavior, and provider acknowledgement boundary.

## Bulk Dispatch

Starting in v7, `Dispatcher@cbq.bulkDispatch` accepts `batchSize` to opt in to chunked enqueueing. This is separate from [tracked batches](batched-jobs.md): bulk dispatch publishes jobs without creating a batch record or lifecycle jobs.

| Argument | Required | Default | Description |
| --- | --- | --- | --- |
| `jobs` | Yes | | Array of Job instances |
| `connectionName` | No | Configured default connection | Connection used for every job in this call |
| `queueName` | No | Each job's queue, then the connection's default queue | Explicit queue overrides every job's queue |
| `batchSize` | No | `1` | Integer from `1` to `100` |

```cfscript
transaction {
    getInstance( "Dispatcher@cbq" ).bulkDispatch(
        jobs = jobs,
        connectionName = "db",
        queueName = "mail",
        batchSize = 100
    );
}
```

The default preserves the existing order: announce `onCBQJobAdded`, reset the job's attempt, and push that job before announcing the next. With a larger chunk, each job is still announced separately, but all added events in that chunk run before it is persisted. Keep the default if an added interceptor must read earlier jobs from the same chunk. Bulk dispatch does not use a job's backoff as an initial delay.

DBProvider inserts at most 100 rows (500 bindings) per statement; each job keeps its own payload, queue row, and lifecycle. Other providers use their normal `push` method unless they implement `pushMany`. The caller owns transaction boundaries. Use a transaction shared with related database changes if all chunks must commit or roll back together. Without it, a later event, serialization, or insert failure can leave earlier chunks persisted. Database transactions do not make an external provider transactional.

In 7.0.2+, publication-result hooks cover the final partial chunk. See [Queue Observability](../observability.md#publishing) for what a result means when a chunk fails.
