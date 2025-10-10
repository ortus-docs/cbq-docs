# Upgrade Guide

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
