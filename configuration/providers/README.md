# Providers

A Queue Provider provides a way to serialize jobs to a Queue Connection and to work jobs off a Queue Connection.

The three built-in cbq providers are:

* [SyncProvider@cbq](syncprovider.md)
* [ColdBoxAsyncProvider@cbq](coldboxasyncprovider.md)
* [DBProvider@cbq](dbprovider.md)

### Writing your own Custom Provider

All Queue Providers extend the `AbstractQueueProvider` base component.  They must implement three abstract methods:

```cfscript

/**
 * Persists a serialized job to the Queue Connection
 *
 * @queueName The queue name for the job.
 * @job       The Job to serialize and push to the queue.
 * @delay     The delay (in seconds) before working the job.
 * @attempts  The current attempt number.
 *
 * @return    AbstractQueueProvider
 */
public any function push(
    required string queueName,
    required AbstractJob job,
    numeric delay = 0,
    numeric attempts = 0
);

/**
 * Starts a worker for a Worker Pool on this Queue Connection.
 *
 * @pool    The Worker Pool that is working this Queue Connection.
 *
 * @return  A function that when called will stop this worker.
 */
public function function startWorker( required WorkerPool pool );

/**
 * Starts any background processes needed for the Worker Pool.
 *
 * @pool    The Worker Pool that is working this Queue Connection.
 *
 * @return  AbstractQueueProvider
 */
public any function listen( required WorkerPool pool );
```

### Optional Bulk Publishing

Starting in v7, a provider may implement `public any function pushMany( required array entries )` for efficient enqueueing. Each entry contains the normal `push` arguments: `queueName`, `job`, and optionally `delay` and `attempts`. The connection falls back to calling `push` for each entry when this method is absent.

Use the same serialization, routing, datasource, and query options as `push`. DBProvider bounds inserts to 100 rows per statement, including direct calls to `pushMany`. In 7.0.2+, the final partial chunk also uses `pushMany`, even if it contains only one entry. The caller controls transaction boundaries; a provider that partially publishes and then throws must not imply that the entire chunk was rolled back.

See [Bulk Dispatch](../../jobs/dispatching-a-job.md#bulk-dispatch) for event ordering and [Queue Observability](../../observability.md#publishing) for publication-result hooks.
