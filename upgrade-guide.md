# Upgrade Guide

## v5 to v6

### Batch table migrations

Copy the following migrations from the installed module's `resources/database/migrations` directory into your application's migration directory and run them with your existing cfmigrations or CommandBox Migrations workflow before starting cbq 6 workers:

| Migration | Change | Applies to |
| --- | --- | --- |
| `2000_01_01_000008_add_successfulJobs_to_cbq_batches_table.cfc` | Adds `successfulJobs`, an unsigned integer with a default of `0` | Applications using batches, with any queue provider |
| `2000_01_01_000009_make_cbq_batches_name_nullable.cfc` | Makes `name` nullable | Applications using batches |

The supplied migrations target `cbq_batches`. If you configured `batchRepositoryProperties.tableName`, adapt the table name in your copied migrations. If you do not use batches, these batch migrations are not needed.

For a fresh installation, apply the initial table migrations and all subsequent migrations for the tables you use, in filename order. Installing the package does not migrate your database automatically.

The success-counter migration defaults existing rows to `0`; it does not reconstruct historical successes. Drain active batches before upgrading when their existing counts need to remain accurate. A custom batch repository must store, increment, and expose `successfulJobs` and decrement `pendingJobs` for both successful and terminally failed jobs. See [batch result accounting](jobs/batched-jobs.md#result-accounting).

v5.0.8 and v5.0.9 also require `successfulJobs`. If you already applied the migration, keep its migration history and do not apply it again.

### Database requirements and retries

For MySQL, DBProvider requires **MySQL 8.0 or later** because reservation queries use `FOR UPDATE SKIP LOCKED`. Upgrade MySQL 5.7 before enabling database workers. See [DBProvider](configuration/providers/dbprovider.md#database-compatibility).

Use cbq 6.0.2 or later on the v6 line for the corrected database attempt counter. Each database execution consumes one attempt, including an execution that requests a manual release; releasing that execution does not consume a second attempt. `maxAttempts = 0` continues to mean unlimited attempts.

### Batch Java library

Keep the bundled `cbq/lib` directory in your application's `this.javaSettings.loadPaths` and restart the application after changing Java load paths. Batch IDs require `com.fasterxml.uuid.Generators`. See [Load Java Libraries](getting-started/installation.md#load-java-libraries).

## v4 to v5

The `push` method signature has changed. It now receives the full `AbstractJob` instance instead of the serialized version.  All the built-in providers have been updated to handle this change.  If you have any custom Providers, please update this method signature.

<table><thead><tr><th>Name</th><th>Type</th><th data-type="checkbox">Required</th><th>Default Value</th><th>Description</th></tr></thead><tbody><tr><td>queueName</td><td><code>string</code></td><td>true</td><td></td><td>The queue name for the job.</td></tr><tr><td>job</td><td><code>AbstractJob</code></td><td>true</td><td></td><td>The Job to serialize and push to the queue.</td></tr><tr><td>delay</td><td><code>numeric</code></td><td>false</td><td><code>0</code></td><td>The delay (in seconds) before working the job.</td></tr><tr><td>attempts</td><td><code>numeric</code></td><td>false</td><td><code>0</code></td><td>The current attempt number.</td></tr></tbody></table>

This change was made to give Providers more flexibility over how Job data is serialized.

## v3 to v4

On Batches, the `finally` and `catch` methods have been deprecated in order to support Adobe ColdFusion.

`finally` -> `onComplete`

`catch` -> `onFailure`

If you are running on Lucee or BoxLang, the old methods names will still work, but they may be removed in a future version.  We recommend migrating to the new method names.

## v2 to v3

In v3, the `cbq_failed_jobs` table migrates the `failedDate` column type from a timestamp to a unix timestamp.

A migration file is included in `resources/database/migrations/2000_01_01_000006_use_unix_timestamp_for_failed_job_log_failedDate.cfc`. To run this migration, your failed jobs table will need to be empty. Alternatively, you can write your own migration that converts the timestamp to a unix timestamp. (The logic is different for each database grammar.)
