# ColdBoxAsyncProvider

{% hint style="danger" %}
This provider is not **durable**.

(Pending jobs are not persisted and may be lost if there are server issues between dispatching the job and working the job.)
{% endhint %}

The ColdBoxAsyncProvider runs any jobs dispatched on a background thread using ColdBox's AsyncManager.  Jobs will be worked by the same server that dispatched them.  The number of workers specified translates to the number of threads dedicated to working the jobs.
