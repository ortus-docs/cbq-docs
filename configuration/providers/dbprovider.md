# DBProvider

{% hint style="success" %}
This provider is **durable**.

(Pending jobs are persisted. If there are server issues, the jobs will be worked when the issues are resolved.  Multiple different servers can dispatch to this Queue Connection and multiple different Worker Pools can work these jobs.)
{% endhint %}

{% hint style="success" %}
This provider **supports multiple queues**.

(An array of prioritized queues can be assigned to Worker Pools worked by this provider.)
{% endhint %}

You may use a database as the backing engine for your Queue Connection using `DBProvider@cbq`. cbq depends on qb 13.x for its database queries.

### Database Compatibility

For MySQL, DBProvider requires **MySQL 8.0 or later**. Reservation queries use `FOR UPDATE SKIP LOCKED`, which MySQL 5.7 does not support. cbq does not ship a MySQL 5.7 fallback grammar. Upgrade before enabling database workers; removing the lock clause changes concurrent worker behavior.

Choose a qb grammar appropriate to your database. The locking and reservation behavior matters in addition to basic query support.

### Configuration

Set provider properties on the queue connection. These are the defaults for the basic database settings:

```cfscript
{
    "tableName": "cbq_jobs",
    "datasource": null,
    "queryOptions": {},
    "pollIntervalMilliseconds": 5000
}
```

#### tableName

The name of the table to use when managing jobs.  This table should have the structure provided by the provided database migration file.

#### datasource

The name of the datasource to use when managing jobs. This overrides any datasource provided in the `queryOptions`.

#### queryOptions

A struct of options that will be passed to [`queryExecute`](https://cfdocs.org/queryexecute) when managing jobs.

#### pollIntervalMilliseconds

The delay in milliseconds between completed native database polling passes. Introduced in v7, it defaults to `5000` and must be a positive integer; invalid values throw `cbq.DBProvider.InvalidPollInterval`. It configures each worker watcher on this connection when workers are registered.

```cfscript
newConnection( "db" )
    .setProvider( "DBProvider@cbq" )
    .setProperties( { "pollIntervalMilliseconds": 250 } );
```

Each pass retrieves only as many jobs as the worker pool has available slots. For short jobs, polling can determine backlog drain time. A shorter interval increases idle polling as well, so measure database load, queue age, worker capacity, and transport limits. It does not increase concurrency, change timeouts or retry backoff, or guarantee a delivery time. Restart or re-register workers after changing the setting.

### Jobs Table Structure

Copy the migrations from the installed module's `resources/database/migrations` directory to your application's migration directory and run them with [CommandBox Migrations](https://forgebox.io/view/commandbox-migrations) or [cfmigrations](https://forgebox.io/view/cfmigrations). Apply the migrations for `cbq_jobs` in filename order. The original create-table migration alone does not produce the final schema: subsequent migrations add ownership and terminal-state columns.

For an existing v6 jobs table, apply `2000_01_01_000010_widen_job_attempts.cfc` before starting v7 workers. Batch tables also require `2000_01_01_000011_track_processed_batch_jobs.cfc`; see the [Upgrade Guide](../../upgrade-guide.md#v6-to-v7).

This is the final jobs-table structure for v7. If you create the table yourself, use this structure instead of just the original create-table migration:

```cfscript
schema.create( "cbq_jobs", function( t ) {
    t.bigIncrements( "id" );
    t.string( "queue" );
    t.longText( "payload" );
    t.unsignedBigInteger( "attempts" );
    t.string( "reservedBy" ).nullable();
    t.unsignedBigInteger( "reservedDate" ).nullable();
    t.unsignedBigInteger( "availableDate" );
    t.unsignedBigInteger( "createdDate" );
    t.unsignedBigInteger( "completedDate" ).nullable();
    t.unsignedBigInteger( "failedDate" ).nullable();
    t.unsignedBigInteger( "lastReleasedDate" ).nullable();

    t.index( "queue" );
    t.index( "reservedBy" );
    t.index( [ "completedDate", "failedDate" ] );
} );
```

Adapt table names in your copied migrations when using a custom `tableName`. Installing the package does not run migrations.

### Reservations and Attempts

Workers reserve only as many jobs as their available execution slots allow. cbq can reclaim expired reservations, including rows with a `reservedBy` value but no `reservedDate` after their `availableDate` expires.

The database attempt count increases once when execution starts. A manual release or retry preserves that execution count rather than incrementing it again. `maxAttempts = 0` permits unlimited retries; the v7 bigint column retains counts for long-lived jobs.

Completion, failure, and release updates check the execution's attempt and reservation state. A late callback from a timed-out execution cannot overwrite a queue row that has been reclaimed for another attempt. This protects queue state; a timeout does not prove the underlying worker has stopped, so jobs should still handle repeated external side effects safely.

### Workers with Different Job Mappings

Use separate queues and explicitly configure each worker pool's queues when workers have different job components. A worker that cannot resolve a job's WireBox mapping logs the job ID and pool, releases its fetched reservation without consuming an attempt or marking the job failed, and continues polling. Another worker with the mapping can claim it. Routing prevents incompatible workers from repeatedly claiming the same job.

### Bulk Inserts

Use [Bulk Dispatch](../../jobs/dispatching-a-job.md#bulk-dispatch) with `batchSize` from 1 to 100 to opt in to chunked inserts. DBProvider also exposes `pushMany( entries )` and bounds direct calls to 100 rows per insert. It uses the configured table, datasource, and query options. Queue names and serialized payloads are bound as text, even when they look numeric. Each job remains a separate queue row, and the caller owns transaction boundaries.

### Queue Health Snapshots

In cbq 7.1.0+, `getQueueHealth(queue)` reads pending depth and age, runnable/delayed work and reserved/stale leases. It uses this provider instance's configured `tableName`, datasource and `queryOptions`, including any query timeout. It does not reserve jobs or start workers.

```cfscript
provider = wirebox.getInstance( "Config@cbq" ).getConnection( "db" ).getProvider();
snapshot = provider.getQueueHealth( "mail" );
```

Pass a queue name, an array of names, or `"*"` for all queues. Selections are bound parameters. Completed/failed jobs are excluded; pending depth is runnable plus delayed work and excludes reserved jobs. Unsupported retry/exhaustion counts are not invented.

Use [`QueueHealthAdapter@cbq`](../../queue-health.md) to select the registered connection directly, combine the snapshot with worker-pool activity, or feed a monitoring collector. Database settings stay on the connection/provider.
