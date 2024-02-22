# Creating a Job

After you define your job, you need to create an instance of your job and set the properties before you dispatch it onto your [Queue Connection](../configuration/config-file/#connection).  You can do this is a few different ways.

## Creating a Job Instance

You can create a Job instance using WireBox anywhere in your ColdBox application.

```cfscript
component {

    function index( event, rc, prc ) {
        var job = getInstance( "SendWelcomeEmailJob" );
    }

}
```

{% hint style="warning" %}
Keep in mind that Job instances are transient. Do not inject a Job instance into a Component or Scope that is not transient.  For instance, do not inject a Job instance as a property in a Handler.
{% endhint %}

### Setting Job Properties

Once you have a Job instance, you can set the data for the particular Job by calling the setter methods.

```cfscript
component {

    function index( event, rc, prc ) {
        var job = getInstance( "SendWelcomeEmailJob" );
        job.setEmail( "john@example.com" );
        job.setGreeting( "Welcome!" );
    }

}
```

### setProperties

You can also set all the properties at once using the `setProperties` method.

{% hint style="warning" %}
Using this method will overwrite any previously set properties.
{% endhint %}

```cfscript
component {

    function index( event, rc, prc ) {
        var job = getInstance( "SendWelcomeEmailJob" );
        job.setProperties( {
            "email": "john@example.com",
            "greeting": "Welcome!"
        } );
    }

}
```

### onConnection

You can override the connection for a job by calling the `onConnection` method with the desired connection name.

### setConnection

See [onConnection](creating-a-job.md#onconnection).

### onQueue

The queue name to use for the job. Queue names can be any string you choose, but to be worked a `WorkerPool` must be defined working the same connection and queue name.

### setQueue

See [onQueue](creating-a-job.md#onqueue).

### setBackoff

Sets the amount of time, in seconds, to wait in-between Job attempts — including the initial attempt.

### setDelay

An alias for [setBackoff](creating-a-job.md#setbackoff).

### setTimeout

Sets the amount of time, in seconds, to let a Job run on a worker before marking it as a failed attempt.

### setMaxAttempts

Sets the maximum number of attempts before a Job is marked as failed.

### chain

Sets an array of Jobs to be executed, in order, after this Job successfully executes.

### getMemento

getMemento

## [cbq.job()](../cbq-model.md#job)

You can also create a Job instance using the `cbq` model. (This can be injected into your components using the `cbq@cbq` DSL.)

{% hint style="success" %}
The cbq model is a singleton, so feel free to inject it into any component.
{% endhint %}

```cfscript
component {

    property name="cbq" inject="cbq@cbq";
    
    function index( event, rc, prc ) {
        var job = cbq.job(
            job = "SendWelcomeEmailJob",
            properties = {
                "email": "john@example.com",
                "greeting": "Welcome!"
            },
            chain = [],
            queue = "priority",
            connection = "db",
            backoff = 10,
            timeout = 30,
            maxAttempts = 3
        );
    }

}
```
