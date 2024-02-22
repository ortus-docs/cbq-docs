# Interceptors

## Interception Points

cbq announces the following interception points:

### `onCBQJobAdded`

This is called when a Job is dispatched to a Queue.

### `onCBQJobMarshalled`

This is called when a Job is pulled off the Queue to work.

### `onCBQJobComplete`&#x20;

This is called when a Job successfully finishes executing.

### `onCBQJobException`&#x20;

This is called when encountering an exception when handling a Job.

### `onCBQJobFailed`&#x20;

This is called when a Job is considered failed, after exhausting its `maxAttempts`.

## JobPattern Annotation

{% hint style="warning" %}
To use the `jobPattern` annotation, you must have enabled the `registerJobInterceptorRestrictionAspect` setting in your [module settings](configuration/module-settings.md).
{% endhint %}

Interceptors listening on the cbq interception points listed above can optionally restrict execution to certain Jobs using a `jobPattern` annotation (similar to the `eventPattern` annotation [in ColdBox](https://coldbox.ortusbooks.com/the-basics/interceptors/restricting-execution)).

```cfscript
component {

    function onCBQJobMarshalled( event, data ) jobPattern="SendWelcomeEmailJob" {
        // check if we've hit the email send limits for the month
    }

}
```

This annotation accepts a regex string to check against the Job full name:

```cfscript
component {

    function onCBQJobMarshalled( event, data ) jobPattern="^.*Email.*$" {
        // check if we've hit the email send limits for the month
    }

}
```

{% hint style="info" %}
Jobs that do not match the interception point will send a notice to the `debug` log, if that is turned on for `JobInterceptorRestriction`.
{% endhint %}
