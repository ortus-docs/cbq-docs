# Dispatching a Job

## Dispatching

Creating a Job is all well and good, but the Job will not be executed until it is dispatched on to a [Queue Connection](../configuration/config-file/#connection).

If you have a Job instance, you can dispatch it by calling the `dispatch` method.

```cfscript
component {

    function index( event, rc, prc ) {
        getInstance( "SendWelcomeEmailJob" );
            .setEmail( "john@example.com" );
            .setGreeting( "Welcome!" )
            .dispatch();
    }

}
```

You can also create and dispatch a job in the same call using the `cbq.dispatch()` method.

```cfscript
component {

    property name="cbq" inject="cbq@cbq";
    
    function index( event, rc, prc ) {
        cbq.dispatch(
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

## Interception Points

### onCBQJobAdded

This is fired before serializing a Job and sending it to a connection.  The data includes the `job` to be serialized and dispatched and the connection the Job is being dispatched on.

```cfscript
variables.interceptorService.announce( "onCBQJobAdded", {
    "job" : job,
    "connection" : connection
} );
```
