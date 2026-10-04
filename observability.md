# Queue Observability

{% hint style="info" %}
These hooks were introduced in [cbq 7.0.1](https://github.com/coldbox-modules/cbq/releases/tag/v7.0.1). They are not available in cbq 5.x.
{% endhint %}

cbq exposes observation points for publishing and individual processing attempts. They let an application record queue timings, correlate retries, and establish tracing context on the actual worker. cbq does not start a tracing client or persist trace headers automatically; those responsibilities belong to your integration.

## Publishing

`onCBQJobAdded` runs before the provider is called. Use it to attach any serializable trace metadata your application needs, but do not treat it as confirmation that enqueueing succeeded.

| Hook | When it runs | Data |
| --- | --- | --- |
| `onCBQJobPublished` | After the provider's push returns successfully | `job`, `connection`, nullable `exception` |
| `onCBQJobPublishException` | When the provider's push throws | `job`, `connection`, original `exception` |

Both single dispatch and bulk dispatch announce these result hooks. Bulk dispatch announces a result for each job in a complete attempted chunk. In 7.0.1, the final partial chunk bypasses the result hooks; use the default `batchSize = 1` when every publication must be observed. [cbq PR #40](https://github.com/coldbox-modules/cbq/pull/40) adds result hooks for that final partial chunk; that fix is not included in 7.0.1. If a chunk throws, cbq rethrows the original provider exception. Earlier chunks can already have been published, and the result hook does not guarantee atomic publication or durable broker acknowledgement beyond the provider's own contract.

With `SyncProvider`, job execution happens inside the push. Its publish-result hook therefore follows synchronous execution; it does not represent acceptance by an external queue.

## Attempt lifecycle

Every attempt receives an `executionId`. It stays the same across that attempt's observation hooks, including callbacks on other threads. A retry gets a new ID. Use this ID to track an attempt; a job ID alone cannot distinguish retries or overlapping work after a timeout.

| Hook | Boundary | Data |
| --- | --- | --- |
| `onCBQJobAttemptScheduled` | Before dispatching execution to the worker | `job`, `executionId`, `attempt` |
| `onCBQJobExecutionStarted` | On the executing thread, before provider/job pre-execution callbacks and `handle()` | `job`, `executionId`, `attempt` |
| `onCBQJobExecutionExited` | In the executing thread's `finally`, after that execution body exits | `job`, `executionId`, `attempt` |
| `onCBQJobAttemptFinished` | In the result/error path, which can run on another thread | `job`, `executionId`, `attempt`, `status` |

The execution bracket covers `beforeJobRun`, the job's `before`, `onCBQJobMarshalled`, and `handle()`. Completion/recovery callbacks run outside that bracket for asynchronous providers. Do not use it to time the full chain, batch, or persistence lifecycle.

`status` is one of:

| Value | Meaning |
| --- | --- |
| `ok` | The successful-result path ran without a requested manual release |
| `released` | The job requested a manual release |
| `internal_error` | The exception path ran |
| `cancelled` | The exception path ran with the job marked cancelled |
| `deadline_exceeded` | The asynchronous exception path identified a timeout |

This is an attempt observation, not a replacement for `onCBQJobFailed`. A timeout or exception can lead to retry rather than final failure. The finished payload does not include the exception; use the existing `onCBQJobException`/`onCBQJobFailed` hooks when your integration needs it.

## Thread boundaries and completion

For normal asynchronous execution, the worker exits before the completion callback runs. With `SyncProvider`, the successful finished observation runs before the execution-exit hook. On an asynchronous timeout, the finished observation can arrive before the underlying worker exits. Timeout completion does not prove that `handle()` has stopped; the worker can continue and exit later.

An integration should:

1. Create an explicit attempt/span handle at `onCBQJobAttemptScheduled`, keyed by `executionId` in a thread-safe store.
2. At `onCBQJobExecutionStarted`, establish the propagated context on that executing thread and retain its previous scope.
3. At `onCBQJobAttemptFinished`, finish that explicit handle with an atomic, idempotent operation. Do not rely on the callback thread's active span.
4. At `onCBQJobExecutionExited`, restore the previous scope on the executing thread, including when the attempt timed out earlier. Keep cleanup state until this hook runs.

The result path can subsequently encounter a completion or recovery exception and announce another finished observation. Consumers must tolerate repeat notifications for the same ID. Decide how to handle a later failure without reopening a completed span, and never let late success overwrite a recorded timeout.

## Installing an interceptor

Register an interceptor in your ColdBox configuration. The following CFML component logs only the observation name and technical context; it can run under CFML or BoxLang's CFML compatibility module.

```cfscript
component {
    property name="log" inject="logbox:logger:{this}";

    function onCBQJobAttemptScheduled( event, data ) {
        variables.log.info( "Queue attempt scheduled", {
            executionId: data.executionId,
            attempt: data.attempt
        } );
    }

    function onCBQJobExecutionStarted( event, data ) {
        variables.log.info( "Queue execution started", { executionId: data.executionId } );
    }

    function onCBQJobAttemptFinished( event, data ) {
        variables.log.info( "Queue attempt finished", {
            executionId: data.executionId,
            status: data.status
        } );
    }

    function onCBQJobExecutionExited( event, data ) {
        variables.log.info( "Queue execution exited", { executionId: data.executionId } );
    }
}
```

For native BoxLang, use a `.bx` class with the same methods and injection:

```boxlang
class {
    property name="log" inject="logbox:logger:{this}";

    function onCBQJobAttemptFinished( event, data ) {
        variables.log.info( "Queue attempt finished", {
            executionId: data.executionId,
            status: data.status
        } )
    }
}
```

These examples illustrate the interceptor API; they do not implement trace propagation or an attempt store. The [`jobPattern` annotation](interceptors.md#jobpattern-annotation) can restrict all six new observation hooks when its module setting is enabled.

## Observer failures and safe telemetry

Exceptions thrown by observers of the six new hooks are caught and reported through a LogBox warning. If logging also fails, cbq writes a short stderr diagnostic containing the observation stage and logging exception type. Observer/diagnostic failures do not replace enqueue errors, change successful publication, or prevent the execution-exit observation.

The existing interception points retain their behavior. In particular, `onCBQJobAdded` and other established hooks are not covered by this new isolation policy; do not use them as failure-proof telemetry callbacks.

Allowlist technical fields such as execution ID, attempt, job mapping, and queue name. Do not export `job.getMemento()`, raw payloads, credentials, or personal record contents. If you store trace headers or enqueue timestamps on a job, use a namespaced, serializable property that survives that provider's serialization. `providerContext` belongs to providers and is not a portable persistence mechanism for tracing metadata.
