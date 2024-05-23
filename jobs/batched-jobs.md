# Batched Jobs

cbq allows for tracking jobs as part of a batch.  The main use case for batched jobs is to dispatch additional jobs when a batch has completed successfully, completed with failures, or completed in any fashion. (Think `try`, `catch`, `finally`.)

## Setup

To utilize batches, you first need to set up a Batch repository.  Batches are tracked separate from jobs. cbq utilizes a database repository to track batches regardless of what provider your Job's connection uses.

A database migration is provided in `resources/database/migrations/2000_01_01_000001_create_cbq_batches_table.cfc`. You can copy this to your own project to run via [cfmigrations](https://forgebox.io/view/cfmigrations) or you can reference the migration or SQL scripts [below](batched-jobs.md#migrations-and-sql-scripts).

You can customize the batch repository using the `batchRepositoryProperties` of cbq's `moduleSettings`.  This is a struct with two properties you can set:

#### tableName

This defaults to `cbq_batches`.  You can set this to any unique table name in your datasource.

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

The primary use case of batches is to dispatch lifecycle jobs when the batch has completed and if the batch completed successfully or completed with failures.  These jobs can be configured using the `then`, `catch`, and `finally` methods.

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

### catch

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
.catch( cbq.job( "ImportCsvFailedJob" ) );
```

### finally

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
.finally( cbq.job( "ImportCsvCompletedJob" ) );
```

## Dispatching Batches

A `PendingBatch` must be dispatched before any of the jobs contained in it are dispatched. This is done using the `dispatch` method on the `PendingBatch`.

### dispatch

Dispatches a PendingBatch and all the Jobs it contains.

<table><thead><tr><th>Arguments</th><th width="156">Type</th><th width="40">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return:** A `Batch` instance created from this `PendingBatch`.

## Interacting with Batches



## Migrations and SQL Scripts

### cfmigrations

```cfscript
schema.create( "cbq_batches", function ( t ) {
    t.string( "id" ).primaryKey();
    t.string( "name" );
    t.unsignedInteger( "totalJobs" );
    t.unsignedInteger( "pendingJobs" );
    t.unsignedInteger( "failedJobs" );
    t.text( "failedJobIds" ); // JSON column
    t.text( "options" ).nullable(); // JSON column
    t.datetime( "createdDate" );
    t.datetime( "cancelledDate" ).nullable();
    t.datetime( "completedDate" ).nullable();
} );
```

### MySQL

```sql
CREATE TABLE `cbq_batches` (
    `id` VARCHAR(255) NOT NULL,
    `name` VARCHAR(255) NOT NULL,
    `totalJobs` INTEGER UNSIGNED NOT NULL,
    `pendingJobs` INTEGER UNSIGNED NOT NULL,
    `failedJobs` INTEGER UNSIGNED NOT NULL,
    `failedJobIds` TEXT NOT NULL,
    `options` TEXT,
    `createdDate` DATETIME NOT NULL,
    `cancelledDate` DATETIME,
    `completedDate` DATETIME,
    CONSTRAINT `pk_cbq_batches_id` PRIMARY KEY (`id`)
)
```

### SQL Server

```sql
CREATE TABLE [cbq_batches] (
    [id] VARCHAR(255) NOT NULL,
    [name] VARCHAR(255) NOT NULL,
    [totalJobs] INTEGER NOT NULL,
    [pendingJobs] INTEGER NOT NULL,
    [failedJobs] INTEGER NOT NULL,
    [failedJobIds] VARCHAR(MAX) NOT NULL,
    [options] VARCHAR(MAX),
    [createdDate] DATETIME2 NOT NULL,
    [cancelledDate] DATETIME2,
    [completedDate] DATETIME2,
    CONSTRAINT [pk_cbq_batches_id] PRIMARY KEY ([id])
)
```

### Postgres

```sql
CREATE TABLE "cbq_batches" (
    "id" VARCHAR(255) NOT NULL,
    "name" VARCHAR(255) NOT NULL,
    "totalJobs" INTEGER NOT NULL,
    "pendingJobs" INTEGER NOT NULL,
    "failedJobs" INTEGER NOT NULL,
    "failedJobIds" TEXT NOT NULL,
    "options" TEXT,
    "createdDate" TIMESTAMP NOT NULL,
    "cancelledDate" TIMESTAMP,
    "completedDate" TIMESTAMP,
    CONSTRAINT "pk_cbq_batches_id" PRIMARY KEY ("id")
)
```

### Oracle

```sql
CREATE TABLE "CBQ_BATCHES" (
    "ID" VARCHAR2(255) NOT NULL,
    "NAME" VARCHAR2(255) NOT NULL,
    "TOTALJOBS" NUMBER(10, 0) NOT NULL,
    "PENDINGJOBS" NUMBER(10, 0) NOT NULL,
    "FAILEDJOBS" NUMBER(10, 0) NOT NULL,
    "FAILEDJOBIDS" CLOB NOT NULL,
    "OPTIONS" CLOB,
    "CREATEDDATE" DATE NOT NULL,
    "CANCELLEDDATE" DATE,
    "COMPLETEDDATE" DATE,
    CONSTRAINT "PK_CBQ_BATCHES_ID" PRIMARY KEY ("ID")
)
```

### SQLite

```sql
CREATE TABLE "cbq_batches" (
    "id" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    "totalJobs" INTEGER NOT NULL,
    "pendingJobs" INTEGER NOT NULL,
    "failedJobs" INTEGER NOT NULL,
    "failedJobIds" TEXT NOT NULL,
    "options" TEXT,
    "createdDate" DATETIME NOT NULL,
    "cancelledDate" DATETIME,
    "completedDate" DATETIME,
    PRIMARY KEY ("id")
)
```
