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
 * @payload   The serialized job string.
 * @delay     The delay (in seconds) before working the job.
 * @attempts  The current attempt number.
 *
 * @return    AbstractQueueProvider
 */
public any function push(
    required string queueName,
    required string payload,
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
