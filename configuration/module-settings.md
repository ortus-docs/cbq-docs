# Module Settings

## Full Module Settings

```cfscript
settings = {
    // The path the custom config file to register connections and worker pools
    "configPath" : "config.cbq",

    // Flag if workers should be registered.
    // If your application only pushes to the queues, you can set this to `false`.
    "registerWorkers" : getSystemSetting( "CBQ_REGISTER_WORKERS", true ),

    // The interval to poll for changes to the worker pool scaling.
    // Defaults to 0 which turns off the scheduled scaling feature.
    "scaleInterval" : 0,

    // The default amount of time, in seconds, to delay a job.
    // Used if the connection and job doesn't define their own.
    "defaultWorkerBackoff" : 0,

    // The default amount of time, in seconds, to wait before timing out a job.
    // Used if the connection and job doesn't define their own.
    "defaultWorkerTimeout" : 60,

    // The default amount of attempts to try before failing a job.
    // Used if the connection and job doesn't define their own.
    "defaultWorkerMaxAttempts" : 1,

    // Datasource information for tracking batches.
    "batchRepositoryProperties" : {
        "tableName" : "cbq_batches",
	"datasource" : "", // `datasource` can also be a struct
	"queryOptions" : {} // The sibling `datasource` property overrides any defined datasource in queryOptions.
    },

    // Flag to turn on logging failed jobs to a database table.
    "logFailedJobs" : false,

    // Datasource information for loggin failed jobs.
    "logFailedJobsProperties" : {
        "tableName" : "cbq_failed_jobs",
	"datasource" : "", // `datasource` can also be a struct.
	"queryOptions" : {} // The sibling `datasource` property overrides any defined datasource in `queryOptions`.
    },
    
    // Flag to allow restricting Job interceptor execution using a `jobPattern` annotation.
    "registerJobInterceptorRestrictionAspect" : false
};
```

### configPath

The configPath is a dot-delimited path to your cbq Config Component.  By convention, this should be placed in your application's `config/` folder alongside other config files like `ColdBox.cfc` and `WireBox.cfc`.

### registerWorkers

This flag is responsible for spinning up Worker Pools when the application starts.  If a particular instance of your application should **not** work jobs as well as dispatch them, then this setting should be set to `false`.  To make this easy, you can set the `CBQ_REGISTER_WORKERS` environment variable and it will be picked up.  (If you override this setting in your own `moduleSettings` it will still take precedence over the environment variable.)

### scaleInterval

{% hint style="danger" %}
This feature is not implemented yet.
{% endhint %}

This is the interval in seconds that the scale job is ran in the background.  The scale job enables you to scale Worker Pools up or down based on any other factors in your application.

**Setting this value to `0` disables the job entirely.**

### defaultWorkerBackoff

This setting provides a default backoff value for a Worker Pool.  This can still be overridden on a configured Worker Pool in your cbq config file or on an individual Job or `dispatch` call.

### defaultWorkerTimeout

This setting provides a default timeout value for a Worker Pool.  This can still be overridden on a configured Worker Pool in your cbq config file or on an individual Job or `dispatch` call.

### defaultWorkerMaxAttempts

This setting provides a default max attempts value for a Worker Pool.  This can still be overridden on a configured Worker Pool in your cbq config file or on an individual Job or `dispatch` call.

### batchRepositoryProperties

A struct of configuration properties for a Batch Repository.  This is only needed if you dispatch any batches.

#### tableName

The name of the batch table. The default is `cbq_batches`.

#### datasource

The datasource to use to interact with the batch table.  If no datasource is provided, it uses the default application datasource.  A `struct` can also be provided if your CFML engine supports it.

#### queryOptions

A struct of query options to pass to the `queryExecute` call when interacting with the batch table.  If a `datasource` is defined above, it will override any `datasource` key inside the `queryOptions`.

### logFailedJobs

This flag will send failed jobs to a configured database table when true.  In some systems, this is called a Dead Letter Queue or DLQ.

### logFailedJobsProperties

#### tableName

The name of the failed jobs table. The default is `cbq_failed_jobs`.

#### datasource

The datasource to use to interact with the failed jobs table.  If no datasource is provided, it uses the default application datasource.  A `struct` can also be provided if your CFML engine supports it.

#### queryOptions

A struct of query options to pass to the `queryExecute` call when interacting with the failed jobs table.  If a `datasource` is defined above, it will override any `datasource` key inside the `queryOptions`.

### registerJobInterceptorRestrictionAspect

Flag to allow [restricting Job interceptor execution](../interceptors.md#jobpattern-annotation) using a `jobPattern` annotation.
