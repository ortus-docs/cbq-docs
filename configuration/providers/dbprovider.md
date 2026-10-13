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
    "queryOptions": {}
}
```

#### tableName

The name of the table to use when managing jobs.  This table should have the structure provided by the provided database migration file.

#### datasource

The name of the datasource to use when managing jobs. This overrides any datasource provided in the `queryOptions`.

#### queryOptions

A struct of options that will be passed to [`queryExecute`](https://cfdocs.org/queryexecute) when managing jobs.

### Jobs Table Structure

Copy the migrations from the installed module's `resources/database/migrations` directory to your application's migration directory and run them with [CommandBox Migrations](https://forgebox.io/view/commandbox-migrations) or [cfmigrations](https://forgebox.io/view/cfmigrations). Apply the migrations for `cbq_jobs` in filename order. The original create-table migration alone does not produce the final schema: subsequent migrations add ownership and terminal-state columns.

For an existing v5 jobs table, v6 does not add a new jobs-table column. Batch tables have separate required migrations; see the [Upgrade Guide](../../upgrade-guide.md#v5-to-v6).

This is the final jobs-table structure for v6. If you create the table yourself, use this structure instead of just the original create-table migration:

```cfscript
schema.create( "cbq_jobs", function( t ) {
    t.bigIncrements( "id" );
    t.string( "queue" );
    t.longText( "payload" );
    t.unsignedTinyInteger( "attempts" );
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

In cbq 6.0.2+, the database attempt count increases once when execution starts. A manual release or retry preserves that execution count rather than incrementing it again. `maxAttempts = 0` permits unlimited retries, but the v6 `attempts` column remains a tiny integer; v7 includes a migration to widen it.
