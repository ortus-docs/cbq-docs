# Worker Pool

A Queue Connection is defined inside your cbq config file using a `QueueConnectionDefinition` builder component.  You can create one of these builder components using the [`newConnection`](./#newconnection) method.

### WorkerPoolDefinition Methods

#### setName

Sets the name for the Worker Pool.  Usually not called directly as the `newWorkerPool` method requires a `name`.

```cfscript
newWorkerPool( "default" )
    .setName( "not-default" );
```

#### forConnection

Sets the name of the associated Connection for the Worker Pool. This must reference an already registered Connection.

```cfscript
newConnection( "db" )
    .provider( "DBProvider@cbq" );

newWorkerPool( "db-worker" )
    .forConnection( "db" );
```

#### setConnectionName

Alias for [forConnection](worker-pool.md#forconnection).

#### quantity

Sets the quantity of workers for this Worker Pool.

```cfscript
newWorkerPool( "default" )
    .forConnection( "db" )
    .quantity( 3 );
```

#### setQuantity

Alias for [quantity](worker-pool.md#quantity).

#### onQueue

The name of a queue to work. The default queue is named `default`.

```cfscript
newWorkerPool( "premium-only" )
    .forConnection( "db" )
    .onQueue( "premium" );
    
newWorkerPool( "priority" )
    .forConnection( "db" )
    .onQueue( "priority" )
    .quantity( 4 );
    
newWorkerPool( "default" )
    .forConnection( "db" );
    // uses `default` queue
```

#### setQueue

Alias for [onQueue](worker-pool.md#onqueues).

#### backoff

Sets the backoff time amount, in seconds.

```cfscript
newWorkerPool( "default" )
    .forConnection( "db" )
    .backoff( 30 );
```

#### setBackoff

Alias for [backoff](worker-pool.md#backoff).

#### timeout

Sets the timeout time amount, in seconds.

```cfscript
newWorkerPool( "default" )
    .forConnection( "db" )
    .timeout( 60 );
```

#### setTimeout

Alias for [timeout](worker-pool.md#timeout).

#### maxAttempts

Sets the max number of attempts.

```cfscript
newWorkerPool( "default" )
    .forConnection( "db" )
    .maxAttempts( 5 );
```

#### setMaxAttempts

Alias for [maxAttempts](worker-pool.md#maxattempts).
