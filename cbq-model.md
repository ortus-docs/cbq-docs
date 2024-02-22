---
description: >-
  This model is provided to make certain tasks easier when defining and
  dispatching jobs, chains, and batches.
---

# cbq Model

### dispatch

Dispatches a job or chain of jobs.

<table><thead><tr><th>Arguments</th><th width="147">Type</th><th>Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job | array&#x3C;Job></td><td><code>true</code></td><td></td><td>A Job instance or a WireBox ID of a Job instance. If the WireBox ID doesn't exist, a <code>NonExecutableJob</code> instance will be used instead. This allows you to dispatch a Job from a server where that Job component is not defined. If an array is passed, a Job Chain is created and dispatched.</td><td></td></tr><tr><td>properties</td><td>struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance.</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job.</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to.</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on.</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs.</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring.</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed.</td><td></td></tr></tbody></table>

**Return:** The dispatched Job instance.

```cfscript
cbq.dispatch(
    job = "SendWelcomeEmailJob",
    properties = { "body": "first body" },
    queue = "default"
);
```

### job

Creates a job or chain of jobs to be dispatched.

<table><thead><tr><th>Arguments</th><th width="147">Type</th><th>Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>job</td><td>string | Job | array&#x3C;Job></td><td><code>true</code></td><td></td><td>A Job instance or a WireBox ID of a Job instance. If the WireBox ID doesn't exist, a <code>NonExecutableJob</code> instance will be used instead. This allows you to dispatch a Job from a server where that Job component is not defined. If an array is passed, a Job Chain is created and dispatched.</td><td></td></tr><tr><td>properties</td><td>struct</td><td><code>false</code></td><td><code>{}</code></td><td>A struct of properties for the Job instance.</td><td></td></tr><tr><td>chain</td><td>array&#x3C;Job></td><td><code>false</code></td><td><code>[]</code></td><td>An array of Jobs to chain after this Job.</td><td></td></tr><tr><td>queue</td><td>string</td><td><code>false</code></td><td></td><td>The queue the Job belongs to.</td><td></td></tr><tr><td>connection</td><td>string</td><td><code>false</code></td><td></td><td>The Connection to dispatch the Job on.</td><td></td></tr><tr><td>backoff</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait between attempting Jobs.</td><td></td></tr><tr><td>timeout</td><td>numeric</td><td><code>false</code></td><td></td><td>The amount of time, in seconds, to wait before treating a Job as erroring.</td><td></td></tr><tr><td>maxAttempts</td><td>numeric</td><td><code>false</code></td><td></td><td>The maximum amount of attempts of a Job before treating the Job as failed.</td><td></td></tr></tbody></table>

**Return:** The new Job instance.

```cfscript
cbq.job( "SendWelcomeEmailJob" )
    .setProperties( { "body": "first body" } )
    .onQueue( "default" )
    .dispatch();
```

### chain

Creates a chain of jobs to be ran.

Alias for calling `firstJob.chain( otherJobs )`.

<table><thead><tr><th>Arguments</th><th width="156">Type</th><th width="40">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>jobs</td><td>array&#x3C;Job></td><td><code>true</code></td><td></td><td>The array of jobs to run in order in a chain.</td><td></td></tr></tbody></table>

**Return:** The first job of the chain with the chained jobs configured to be dispatched.

```cfscript
cbq.chain( [
    cbq.job( "SendWelcomeEmailJob", { "body": "One" }, [], "default" ),
    cbq.job( "SendWelcomeEmailJob", { "body": "Two" }, [], "default", "sync" ),
    cbq.job( "SendWelcomeEmailJob", { "body": "Three" }, [], "default" )
] )
.dispatch()
```

### batch

Creates a PendingBatch from the Jobs provided.

{% hint style="warning" %}
To use batches, you must first configure a `BatchRepository`.&#x20;

Learn more in the [Batched Jobs documentation](jobs/batched-jobs.md).
{% endhint %}

<table><thead><tr><th>Arguments</th><th width="123">Type</th><th width="331">Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>jobs</td><td>array&#x3C;Job></td><td><code>true</code></td><td></td><td>An array of jobs to batch together.</td><td></td></tr></tbody></table>

**Return:** The PendingBatch to be dispatched.

```cfscript
var batch = cbq
    .batch( [
        cbq.job( "ImportCsvJob", { "start": 1, "end": 100 } ),
	cbq.job( "ImportCsvJob", { "start": 101, "end": 200 } ),
	cbq.job( "ImportCsvJob", { "start": 201, "end": 300 } ),
	cbq.job( "ImportCsvJob", { "start": 301, "end": 400 } ),
	cbq.job( "ImportCsvJob", { "start": 401, "end": 500 } )
    ] )
    .then( cbq.job( "ImportCsvSuccessfulJob" ) )
    .catch( cbq.job( "ImportCsvFailedJob" ) )
    .finally( cbq.job( "ImportCsvCompletedJob" ) )
    .dispatch();
```
