# Batched Jobs

cbq allows for tracking jobs as part of a batch.  The main use case for batched jobs is to dispatch additional jobs when a batch has completed successfully, completed with failures, or completed in any fashion. (Think `try`, `catch`, `finally`.)

## Setup

To utilize batches, you first need to set up a Batch repository.  Batches are tracked separate from jobs. cbq utilizes a database repository to track batches regardless of what provider your Job's connection uses.

Copy the batch-table migrations from `resources/database/migrations` to your own project and run them in filename order with [cfmigrations](https://forgebox.io/view/cfmigrations). The initial `2000_01_01_000001_create_cbq_batches_table.cfc` migration needs the later batch migrations described in [Result Accounting](#result-accounting). Alternatively, create the final structure using the migration or SQL scripts [below](batched-jobs.md#migrations-and-sql-scripts).

You can customize the batch repository using the `batchRepositoryProperties` of cbq's `moduleSettings`.  This is a struct with the following properties:

#### tableName

This defaults to `cbq_batches`.  You can set this to any unique table name in your datasource.

#### datasource

The datasource used for the batch table. This overrides a datasource supplied in `queryOptions`.

#### queryOptions

This is a struct of options that will be passed to `queryExecute`.  The most common use case for this property is to specify a specific datasource to use for the batches table. This defaults to an empty struct (`{}`).

## Defining Batches

To create a batch, you can get an instance of `PendingBatch@cbq` from WireBox or use the [`cbq.batch()`](../cbq-model.md#batch) helper method.

Once you have a `PendingBatch` instance, you can add jobs to the batch using the `add` method.

### add

Adds a single Job or an array of Jobs to a PendingBatch.

<table><thead><tr><th>Arguments</th><th width="123">Type</th><th width="331">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job | array&#x3C;Job></td><td><code>true</code></td><td></td><td>The Job WireBox id, Job instance, or array of Job instances to add to the PendingBatch.</td><td></td></tr><tr><td>properties</td><td>Struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed. (Only used when providing a Job WireBox id.)</td><td></td></tr></tbody></table>

**Return:** The PendingBatch to be dispatched.

```cfscript
batch
    .add( cbq.job( "ImportCsvJob", { "start": 1, "end": 100 } ) )
    .add( "ImportCsvJob", { "start": 101, "end": 200 } )
    .add( [
        cbq.job( "ImportCsvJob", { "start": 201, "end": 300 } ),
	cbq.job( "ImportCsvJob", { "start": 301, "end": 400 } ),
	cbq.job( "ImportCsvJob", { "start": 401, "end": 500 } )
    ] );
```

The primary use case of batches is to dispatch lifecycle jobs when the batch has completed and if the batch completed successfully or completed with failures.  These jobs can be configured using the `then`, `onFailure`, and `onComplete` methods. Use the latter names for Adobe ColdFusion compatibility; `catch` and `finally` are deprecated aliases on Lucee and BoxLang.

### then

Defines a Job to be dispatched when all the jobs in the batch finishes successfully.

<table><thead><tr><th>Arguments</th><th width="123">Type</th><th width="331">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job</td><td><code>true</code></td><td></td><td>The Job WireBox id or Job instance to execute if all the jobs in the Batch complete successfully.</td><td></td></tr><tr><td>properties</td><td>Struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed. (Only used when providing a Job WireBox id.)</td><td></td></tr></tbody></table>

**Return:** The `PendingBatch` instance.

```cfscript
cbq.batch( [
    cbq.job( "ImportCsvJob", { "start": 1, "end": 100 } ),
    cbq.job( "ImportCsvJob", { "start": 101, "end": 200 } ),
    cbq.job( "ImportCsvJob", { "start": 201, "end": 300 } ),
    cbq.job( "ImportCsvJob", { "start": 301, "end": 400 } ),
    cbq.job( "ImportCsvJob", { "start": 401, "end": 500 } )
] )
.then( cbq.job( "ImportCsvSuccessfulJob" ) );
```

### onFailure

Defines a Job to be dispatched the first time a job in the Batch fails.

<table><thead><tr><th>Arguments</th><th width="123">Type</th><th width="331">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job</td><td><code>true</code></td><td></td><td>The Job WireBox id or Job instance to execute the first time a job in the Batch fails.</td><td></td></tr><tr><td>properties</td><td>Struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed. (Only used when providing a Job WireBox id.)</td><td></td></tr></tbody></table>

**Return:** The `PendingBatch` instance.

```cfscript
cbq.batch( [
    cbq.job( "ImportCsvJob", { "start": 1, "end": 100 } ),
    cbq.job( "ImportCsvJob", { "start": 101, "end": 200 } ),
    cbq.job( "ImportCsvJob", { "start": 201, "end": 300 } ),
    cbq.job( "ImportCsvJob", { "start": 301, "end": 400 } ),
    cbq.job( "ImportCsvJob", { "start": 401, "end": 500 } )
] )
.onFailure( cbq.job( "ImportCsvFailedJob" ) );
```

### onComplete

Defines a Job to be dispatched after all the jobs in the Batch have executed successfully or failed.

<table><thead><tr><th>Arguments</th><th width="123">Type</th><th width="331">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job</td><td><code>true</code></td><td></td><td>The Job WireBox id or Job instance to execute after all the jobs in the Batch have executed successfully or failed.</td><td></td></tr><tr><td>properties</td><td>Struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring. (Only used when providing a Job WireBox id.)</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed. (Only used when providing a Job WireBox id.)</td><td></td></tr></tbody></table>

**Return:** The `PendingBatch` instance.

```cfscript
cbq.batch( [
    cbq.job( "ImportCsvJob", { "start": 1, "end": 100 } ),
    cbq.job( "ImportCsvJob", { "start": 101, "end": 200 } ),
    cbq.job( "ImportCsvJob", { "start": 201, "end": 300 } ),
    cbq.job( "ImportCsvJob", { "start": 301, "end": 400 } ),
    cbq.job( "ImportCsvJob", { "start": 401, "end": 500 } )
] )
.onComplete( cbq.job( "ImportCsvCompletedJob" ) );
```

## Dispatching Batches

A `PendingBatch` must be dispatched before any of the jobs contained in it are dispatched. This is done using the `dispatch` method on the `PendingBatch`.

### dispatch

Dispatches a PendingBatch and all the Jobs it contains.

<table><thead><tr><th>Arguments</th><th width="156">Type</th><th width="40">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return:** A `Batch` instance created from this `PendingBatch`.

## Result Accounting

cbq 6 requires migration `2000_01_01_000008_add_successfulJobs_to_cbq_batches_table.cfc` for `successfulJobs`, and migration `2000_01_01_000009_make_cbq_batches_name_nullable.cfc` makes batch names optional. Apply them to the configured batch table even when your jobs use SyncProvider or ColdBoxAsyncProvider. See the [Upgrade Guide](../upgrade-guide.md#v5-to-v6).

`totalJobs` counts jobs added to the batch. Each successful result increments `successfulJobs`; each terminal failure increments `failedJobs` and adds its ID to `failedJobIds`. Both outcomes decrement `pendingJobs`. A retry is not a terminal result. The `onComplete` job can run when the final pending job fails, and an exception dispatching a `then` or `onFailure` job is logged so the completion path can continue.

## Interacting with Batches

Reload a batch from its repository to read current counts; a previously returned `Batch` instance is a snapshot.

```cfscript
var currentBatch = getInstance( "DBBatchRepository@cbq" ).find( batch.getId() );
var counts = {
    total: currentBatch.getTotalJobs(),
    pending: currentBatch.getPendingJobs(),
    successful: currentBatch.getSuccessfulJobs(),
    failed: currentBatch.getFailedJobs()
};
```

## Migrations and SQL Scripts

The examples below show the batch table after the v6 migrations. For a fresh installation, create this final structure or apply the supplied migrations in filename order. For an existing table, use the upgrade migrations instead of recreating it. Dates are Unix timestamps, matching the repository's stored values.

### cfmigrations

```cfscript
schema.create( "cbq_batches", function ( t ) {
    t.string( "id" ).primaryKey();
    t.string( "name" ).nullable();
    t.unsignedInteger( "totalJobs" );
    t.unsignedInteger( "pendingJobs" );
    t.unsignedInteger( "successfulJobs" ).default( 0 );
    t.unsignedInteger( "failedJobs" );
    t.text( "failedJobIds" ); // JSON column
    t.text( "options" ).nullable(); // JSON column
    t.unsignedInteger( "createdDate" );
    t.unsignedInteger( "cancelledDate" ).nullable();
    t.unsignedInteger( "completedDate" ).nullable();
} );
```

### MySQL

```sql
CREATE TABLE `cbq_batches` (
    `id` VARCHAR(255) NOT NULL,
    `name` VARCHAR(255),
    `totalJobs` INTEGER UNSIGNED NOT NULL,
    `pendingJobs` INTEGER UNSIGNED NOT NULL,
    `successfulJobs` INTEGER UNSIGNED NOT NULL DEFAULT 0,
    `failedJobs` INTEGER UNSIGNED NOT NULL,
    `failedJobIds` TEXT NOT NULL,
    `options` TEXT,
    `createdDate` INTEGER UNSIGNED NOT NULL,
    `cancelledDate` INTEGER UNSIGNED,
    `completedDate` INTEGER UNSIGNED,
    CONSTRAINT `pk_cbq_batches_id` PRIMARY KEY (`id`)
)
```

### SQL Server

```sql
CREATE TABLE [cbq_batches] (
    [id] VARCHAR(255) NOT NULL,
    [name] VARCHAR(255),
    [totalJobs] INTEGER NOT NULL,
    [pendingJobs] INTEGER NOT NULL,
    [successfulJobs] INTEGER NOT NULL DEFAULT 0,
    [failedJobs] INTEGER NOT NULL,
    [failedJobIds] VARCHAR(MAX) NOT NULL,
    [options] VARCHAR(MAX),
    [createdDate] INTEGER NOT NULL,
    [cancelledDate] INTEGER,
    [completedDate] INTEGER,
    CONSTRAINT [pk_cbq_batches_id] PRIMARY KEY ([id])
)
```

### Postgres

```sql
CREATE TABLE "cbq_batches" (
    "id" VARCHAR(255) NOT NULL,
    "name" VARCHAR(255),
    "totalJobs" INTEGER NOT NULL,
    "pendingJobs" INTEGER NOT NULL,
    "successfulJobs" INTEGER NOT NULL DEFAULT 0,
    "failedJobs" INTEGER NOT NULL,
    "failedJobIds" TEXT NOT NULL,
    "options" TEXT,
    "createdDate" INTEGER NOT NULL,
    "cancelledDate" INTEGER,
    "completedDate" INTEGER,
    CONSTRAINT "pk_cbq_batches_id" PRIMARY KEY ("id")
)
```

### Oracle

```sql
CREATE TABLE "CBQ_BATCHES" (
    "ID" VARCHAR2(255) NOT NULL,
    "NAME" VARCHAR2(255),
    "TOTALJOBS" NUMBER(10, 0) NOT NULL,
    "PENDINGJOBS" NUMBER(10, 0) NOT NULL,
    "SUCCESSFULJOBS" NUMBER(10, 0) DEFAULT 0 NOT NULL,
    "FAILEDJOBS" NUMBER(10, 0) NOT NULL,
    "FAILEDJOBIDS" CLOB NOT NULL,
    "OPTIONS" CLOB,
    "CREATEDDATE" NUMBER(10, 0) NOT NULL,
    "CANCELLEDDATE" NUMBER(10, 0),
    "COMPLETEDDATE" NUMBER(10, 0),
    CONSTRAINT "PK_CBQ_BATCHES_ID" PRIMARY KEY ("ID")
)
```

### SQLite

```sql
CREATE TABLE "cbq_batches" (
    "id" TEXT NOT NULL,
    "name" TEXT,
    "totalJobs" INTEGER NOT NULL,
    "pendingJobs" INTEGER NOT NULL,
    "successfulJobs" INTEGER NOT NULL DEFAULT 0,
    "failedJobs" INTEGER NOT NULL,
    "failedJobIds" TEXT NOT NULL,
    "options" TEXT,
    "createdDate" INTEGER NOT NULL,
    "cancelledDate" INTEGER,
    "completedDate" INTEGER,
    PRIMARY KEY ("id")
)
```
