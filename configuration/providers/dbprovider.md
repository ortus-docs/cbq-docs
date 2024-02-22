# DBProvider

{% hint style="success" %}
This provider is **durable**.

(Pending jobs are persisted. If there are server issues, the jobs will be worked when the issues are resolved.  Multiple different servers can dispatch to this Queue Connection and multiple different Worker Pools can work these jobs.)
{% endhint %}

You may use a database as the backing engine for your Queue Connection using the DBProvider.  All database grammars that are [supported by qb](https://qb.ortusbooks.com/v/9.0.0/installation-and-usage) are supported.

### Configuration

The DBProvider has three optional arguments.  The are presented below with their default values.

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

The `cbq_jobs` table must have a specific structure.  It is provided in the form of a migration file called `2000_01_01_000000_create_cbq_jobs_table.cfc`. This migration can be ran via [CommandBox Migrations](https://forgebox.io/view/commandbox-migrations) or [CFMigrations](https://forgebox.io/view/cfmigrations).  If you wish, you can generate the table in other ways, so long as it matches the structure provided in the migration file.

If you choose to use the migration file in your application, copy the file out to your own migrations folder first.

```cfscript
schema.create( "cbq_jobs", function ( t ) {
    t.bigIncrements( "id" );
    t.string( "queue" );
    t.longText( "payload" );
    t.unsignedTinyInteger( "attempts" );
    t.unsignedInteger( "reservedDate" ).nullable();
    t.unsignedInteger( "availableDate" );
    t.unsignedInteger( "createdDate" );

    t.index( "queue" );
} );
```
