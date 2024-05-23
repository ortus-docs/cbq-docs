# SyncProvider

{% hint style="danger" %}
This provider is not **durable**.

(Pending jobs are not saved and may be lost if there are server issues between dispatching the job and working the job.)
{% endhint %}

{% hint style="danger" %}
This provider **does not support multiple queues**.

(Only a single queue can be provided to a Worker Pool instance. Additional queues to be worked should be registered as separate Worker Pool instances.)
{% endhint %}

The SyncProvider runs any jobs dispatched during a request in the same request synchronously. This can be especially useful in development when debugging jobs.  If you job would throw an exception, you will see it with any chosen error handler and tools you use to debug local exceptions.
