# Failed Jobs

## Failed Attempt vs Failed Job

An important distinction to make is between failed Job attempts and failed Jobs.  A Job is attempted up to the defined `maxAttempts`. If a Job fails all of its `maxAttempts` then it is marked as a failed Job and removed from the queue.  Otherwise, the Job is dispatched back to the queue with the current execution count increased.

## LogBox

Failed Job attempts are logged to LogBox.  You will see the exception as well as the serialized Job memento in the `extraInfo`.

## Interceptors

### onCBQJobException

This interception point is fired for every **Job Attempt** failure. The data includes the `job` component and the `exception`.

```cfscript
variables.interceptorService.announce( "onCBQJobException", {
    "job" : job,
    "exception" : e
} );
```

### onCBQJobFailed

This interception point is fired when the Job is marked as failed — when the Job has failed all attempts up to the configured `maxAttempts`. The data includes the `job` component and the `exception`.

```cfscript
variables.interceptorService.announce( "onCBQJobFailed", {
    "job" : job,
    "exception" : e
} );
```

## onFailure Job Method

You can define an `onFailure` method on your Job component.  It will be called with the `exception`.

```cfscript
component extends="cbq.models.Jobs.AbstractJob" {

    function handle() {
        // ...
    }
    
    function onFailure( e ) {
        log.error( "Bad things happened: #e.message#" );
    }

}
```

## Logging Failed Jobs

## Failed Jobs Table

cbq includes an interceptor to log failed jobs to a database.  You can enable this in your module settings:

```cfscript
moduleSettings = {
    "cbq": {
        // Flag to turn on logging failed jobs to a database table.
	"logFailedJobs" : false,
	// Datasource information for loggin failed jobs.
	"logFailedJobsProperties" : {
	    "tableName" : "cbq_failed_jobs",
	    "datasource" : "", // `datasource` can also be a struct.
	    "queryOptions" : {} // The sibling `datasource` property overrides any defined datasource in `queryOptions`.
	}
    }
};
```

Also included is a migration file to add the needed table to your database.  You can find it in `resources/database/migrations/2000_01_01_000002_create_cbq_failed_jobs_table.cfc`. It is also included below.

```cfscript
component {

    function up( schema ) {
        schema.create( "cbq_failed_jobs", function ( t ) {
            t.bigIncrements( "id" );
            t.string( "connection" );
            t.string( "queue" );
            t.string( "mapping" );
            t.longText( "memento" );
            t.longText( "properties" );
            t.string( "exceptionType" ).nullable();
            t.string( "exceptionMessage" );
            t.string( "exceptionDetail" ).nullable();
            t.longText( "exceptionExtendedInfo" ).nullable();
            t.longText( "exceptionStackTrace" );
            t.longText( "exception" );
            t.timestamp( "failedDate" ).withCurrent();
        } );
    }

    function down( schema ) {
        schema.dropIfExists( "cbq_failed_jobs" );
    }

}
```

{% code title="MySQL" %}
```sql
CREATE TABLE `cbq_failed_jobs` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `connection` NVARCHAR(255) NOT NULL,
    `queue` NVARCHAR(255) NOT NULL,
    `mapping` NVARCHAR(255) NOT NULL,
    `memento` LONGTEXT NOT NULL,
    `properties` LONGTEXT NOT NULL,
    `exceptionType` NVARCHAR(255),
    `exceptionMessage` NVARCHAR(255) NOT NULL,
    `exceptionDetail` NVARCHAR(255),
    `exceptionExtendedInfo` LONGTEXT,
    `exceptionStackTrace` LONGTEXT NOT NULL,
    `exception` LONGTEXT NOT NULL,
    `failedDate` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
)
```
{% endcode %}

{% code title="SQL Server" %}
```sql
CREATE TABLE [dbo].[cbq_failed_jobs](
    [id] BIGINT NOT NULL IDENTITY,
    [connection] NVARCHAR(255) NOT NULL,
    [queue] NVARCHAR(255) NOT NULL,
    [mapping] NVARCHAR(255) NOT NULL,
    [memento] NVARCHAR(MAX) NOT NULL,
    [properties] NVARCHAR(MAX) NOT NULL,
    [exceptionType] NVARCHAR(255),
    [exceptionMessage] NVARCHAR(255) NOT NULL,
    [exceptionDetail] NVARCHAR(255),
    [exceptionExtendedInfo] NVARCHAR(MAX),
    [exceptionStackTrace] NVARCHAR(MAX) NOT NULL,
    [exception] NVARCHAR(MAX) NOT NULL,
    [failedDate] DATETIME2 NOT NULL CONSTRAINT [df_cbq_failed_jobs_failedDate] DEFAULT CURRENT_TIMESTAMP
)
```
{% endcode %}

## Retrying a Failed Job

## Configuring Job Backoff
