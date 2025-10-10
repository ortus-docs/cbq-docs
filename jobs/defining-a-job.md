# Defining a Job

A Job represents both the object to be serialized to a [Connection](../configuration/config-file/#connection) to eventually work as well as the code to execute when working a Job.

## Extending from AbstractJob

The first step to define a Job is to extend from `AbstractJob`.  This provides the necessary helper methods to run a Job through the cbq pipeline.

```cfscript
component
    name="SendWelcomeEmailJob"
    extends="cbq.models.Jobs.AbstractJob"
{

}
```

## Properties

Defining properties on your job allow you to see at a glance what data your Job expects when constructing it.  These properties and their values will be serialized to a [Connection](../configuration/config-file/#connection) when dispatching a Job.

{% hint style="info" %}
Defining the properties is not strictly necessary, but your future self will thank you when you try to remember what properties your job is using.
{% endhint %}

```cfscript
component
    name="SendWelcomeEmailJob"
    extends="cbq.models.Jobs.AbstractJob"
{

    property name="email";
    property name="greeting";

}
```

## The handle method

The `handle` method is called when a Job is worked.  Before being called, a Job will be reconstructed with the serialized data from the Connection.  This method is the only required method when defining a Job.

```cfscript
component
    name="SendWelcomeEmailJob"
    extends="cbq.models.Jobs.AbstractJob"
{

    property name="mailService" inject="provider:MailService@cbmailservices";
    
    property name="email";
    property name="greeting";
    
    function handle() {
        variables.mailService.newMail( 
            to = variables.email,
	    from = "noreply@example.com",
	    subject = "Welcome!",
            type = "html",
	    bodyTokens = { 
		"greeting": variables.greeting
		"link": getInstance( "coldbox:requestContext" )
		    .buildLink( "home" )
	    }
        )
        .setView( "_emails/welcome" )
        .send();
    }

}
```

{% hint style="info" %}
Prefer using `provider:` injections or inline `getInstance` calls for logic in your `handle` method.  The `Job` component is created both when dispatching and when working your job, so utilizing these tools will reduce unnecessary processing time when dispatching your job.
{% endhint %}

## Job Execution Properties

A job can define several execution properties on the job itself.  If defined, these values override the module, connection, or worker defaults.  It can still be overridden using the job methods when creating a job.

<pre class="language-cfscript"><code class="lang-cfscript"><strong>component extends="cbq.models.Jobs.AbstractJob" {
</strong>
    // Name of the connection to dispatch this job on.
    variables.connection = "db";
    
    // Name of the queue to use for this job.
    variables.queue = "priority";
    
    // Time, in seconds, between job attempts, including the initial attempt.
    variables.backoff = 15;
    
    // Time, in seconds, to let a job run before failing it.
    variables.timeout = 30;
    
    // Max number of attempts before marking a job as failed.
    // Use 0 for unlimited retries.
    variables.maxAttempts = 9;

}
</code></pre>

## Lifecycle Methods

A Job can define a `before` or `after` method that will be called as part of the Job lifecycle.

<pre class="language-cfscript"><code class="lang-cfscript"><strong>component
</strong>    name="SendWelcomeEmailJob"
    extends="cbq.models.Jobs.AbstractJob"
{

    property name="mailService" inject="provider:MailService@cbmailservices";
    
    property name="email";
    property name="greeting";
    
    function handle() {
        variables.mailService.newMail( 
            to = variables.email,
	    from = "noreply@example.com",
	    subject = "Welcome!",
            type = "html",
	    bodyTokens = { 
		"greeting": variables.greeting
		"link": getInstance( "coldbox:requestContext" )
		    .buildLink( "home" )
	    }
        )
        .setView( "_emails/welcome" )
        .send();
    }
    
    function before() {
        log.debug( "About to execute SendWelcomeEmailJob" );
    }
    
    function after() {
        log.debug( "Finished executing SendWelcomeEmailJob" );
    }

}
</code></pre>

## Manual Control

### release

A job can be manually released from inside a Job.  Releasing a Job sends it back to the queue and increments the attempt count.  An optional delay (in seconds) can be passed to `release`.

```cfscript
component
    name="ProcessFTPDropJob"
    extends="cbq.models.Jobs.AbstractJob"
{
    
    function handle() {
        if ( fileIsNotAvailable() ) {
            this.release( 60 * 60 ); // try again in 1 hour
        } else {
            processFile();
        }
    }

}
```

### cancel

A job can be cancelled from inside a Job.  Cancelling a Job fails the Job and skips any future retries.  It accepts an optional error message as its argument.

```cfscript
component
    name="ProcessFTPDropJob"
    extends="cbq.models.Jobs.AbstractJob"
{
    
    function handle() {
        if ( fileIsNotAvailable() ) {
            this.release( 60 * 60 ); // try again in 1 hour
        } else if ( fileIsCorrupted() ) {
            this.cancel( "File is corrupted" );
        } else {
            processFile();
        }
    }

}
```

## ProviderContext

One field on a job is provided exclusively for Providers — the `providerContext` field.  Providers can use this field to store any data necessary on the `Job` instance itself.  This can be no data, a struct of metadata, or an underlying Java class.  This makes it easier for the provider to get that data back in the Provider lifecycle methods later.

{% hint style="danger" %}
**NOTE:** Do not use this field from inside your Job itself. It is meant only to be set and retrieved from the Provider itself.  Using this field inside your Job may couple your Job to a specific Provider.
{% endhint %}
