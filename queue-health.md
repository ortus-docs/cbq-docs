# Queue Health Snapshots

New in cbq 7.1.0, `QueueHealthAdapter@cbq` returns a read-only snapshot of queue backlog and supported worker activity. It does not start workers, reserve jobs, schedule polling, or send metrics. Call it from your own monitoring code or a collector.

Choose a [queue connection](configuration/config-file/queue-connection.md) for backlog without local workers, or a [worker pool](configuration/config-file/worker-pool.md) for worker activity plus its queues. The adapter has no Sentry dependency.

## Read a queue without workers

```cfscript
adapter = wirebox.getInstance( "QueueHealthAdapter@cbq" );
snapshot = adapter.sample( {
    // cbq's source and queue selection for this sample.
    options: {
        // Registered cbq connection. No worker pool is needed.
        connection: "db",
        // Optional queue selection; omit to use the connection's default queue.
        queue: "mail"
    }
} );
```

For example, a database queue with three runnable jobs, two delayed jobs and one reserved job could return:

```json
{
    "depth": 5,
    "runnable": 3,
    "delayed": 2,
    "reserved": 1,
    "staleReservations": 0,
    "oldestAgeSeconds": 84
}
```

Connection-only snapshots contain no worker fields. Queue counts do not establish whether workers are running on another server.

## Include worker activity

```cfscript
snapshot = adapter.sample( {
    // Select an existing worker pool and its configured queue source.
    options: {
        // Worker pool name, not a connection name or metric label.
        workerPool: "outbound"
    }
} );
```

The adapter resolves the worker pool's actual connection and uses its configured queue selection. It adds `configuredWorkers`, and `activeExecutions` when the executor exposes that measurement. For example, a worker pool configured for four workers with one active execution can report `configuredWorkers: 4` and `activeExecutions: 1`.

Worker measurements describe the **whole worker pool**, even when `options.queue` narrows the backlog selection. Use connection-only samples for extra per-queue breakdowns so the same worker activity is not counted repeatedly.

## Select a source and queues

The public method is `sample(context)`. Put cbq settings in `context.options`; a collector can supply other context fields, but cbq does not use them to choose a connection or worker pool. Each call resolves the selected source again.

| Option | Meaning |
| --- | --- |
| `workerPool` | Registered worker pool name. Its connection selects the provider; its queues are the default selection. |
| `connection` | Registered queue connection name. Its default queue is used when `queue` is omitted. No worker pool is resolved. |
| `queue` | Optional override for the provider's queue selection. |

Set **exactly one** of `workerPool` or `connection`. There is no default source. A missing or blank source name, an array or struct in place of a source name, both selectors together, or legacy `options.pool` throws `cbq.Telemetry.InvalidQueueHealthSelection`. Missing registered connections or worker pools retain cbq's normal lookup errors.

Queue selection follows the provider's capabilities. The database provider accepts a queue name, an array such as `[ "mail", "pdf" ]`, or `"*"` for all queues. An array containing `"*"` also selects all queues. An aggregate snapshot does not preserve worker queue priority.

## Read the available fields

| Field | Meaning |
| --- | --- |
| `configuredWorkers` | Configured quantity for the selected worker pool. |
| `activeExecutions` | Active executor tasks, when the executor supports measuring them. |
| `depth` | Pending, unclaimed jobs. For DBProvider, this is `runnable + delayed`; reserved jobs are separate. |
| `oldestAgeSeconds` | Age since creation of the oldest pending, unclaimed job, including delayed work. |
| `runnable` | Unclaimed jobs available to run now. |
| `delayed` | Unclaimed jobs whose availability time is still in the future. |
| `reserved` | Jobs with a reservation timestamp or owner. |
| `staleReservations` | Reserved jobs whose execution lease has expired. Included in `reserved`, not an additional count. |
| `retries`, `exhausted` | Optional provider counts. DBProvider does not currently return these fields. |

DBProvider excludes completed and failed jobs. A successful empty database snapshot returns measured zeros, including zero age. The adapter does not return job payloads, SQL, identifiers or exception text in a successful sample.

Unsupported measurements stay absent. An unstarted executor does not imply zero activity, and the adapter does not measure worker heartbeats. A provider without `getQueueHealth(queue)` returns only supported worker fields, or `{}` for a connection-only sample. A failed query throws; it does not become a zero backlog. Keep a sampling failure separate from the last successful reading in your collector.

### Database provider settings

[`DBProvider.getQueueHealth(queue)`](configuration/providers/dbprovider.md#queue-health-snapshots) uses that provider instance's configured table, datasource and query options. Queue selections are bound parameters. You do not pass cbq table names or database credentials to the monitoring adapter.

Set any query timeout through the connection's existing `queryOptions`; the snapshot does not add its own timeout or schedule. Custom providers can expose `getQueueHealth(queue)` returning supported numeric fields. The adapter copies only finite, nonnegative queue measurements.

## Use a collector

Call `sample(context)` on your chosen interval and send the returned numeric fields through your monitoring system. cbq does not create a periodic sampler itself.

For a Sentry version that supports `automaticCollectors.queues`, configuration can look like this:

```cfscript
// Sentry owns periodic collection; cbq only supplies the snapshot.
moduleSettings.sentry.automaticCollectors = {
    // Start the sampler. Sentry also needs its valid DSN and metrics enabled.
    enabled: true,
    // One list for provider-owned queue and worker snapshots.
    queues: [ {
        // Fixed metric label; it does not select a cbq source.
        name: "mail",
        // Explicitly choose the adapter shipped by cbq.
        adapter: "QueueHealthAdapter@cbq",
        // Settings passed to this adapter.
        options: {
            // Select worker activity plus backlog through this worker pool.
            workerPool: "outbound"
        }
    } ]
};
```

The collector API is a separate Sentry feature; installing cbq 7.1.0 does not add it to an older Sentry release. Use the direct `sample(context)` calls above with another collector or when that Sentry API is unavailable.

`WorkerHealthAdapter@cbq` remains an alias. Existing manual worker integrations can also call `samplePool(workerPool)` with a worker-pool object to read worker fields only.
