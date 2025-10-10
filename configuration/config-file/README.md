# Config File

The cbq config file is where you define Queue Connections as well as Worker Pools.  These Connections and Worker Pools can also be added, removed, or modified based on the current environment.  When you want to change where your queued Jobs are sent or how they are worked, this is the file you will modify.

## Definitions

### Queue Connections

Also referred to as "Connection."  A Queue Connection defines where Jobs are serialized and the default settings that apply.

## configure

The `configure` method is where the production Queue Connection and Worker Pools definitions are constructed.

### newConnection

This command creates a new `QueueConnectionDefinition` builder component.  It requires a unique name that will define this Connection and that will be used when defining Worker Pools.

<table><thead><tr><th>Arguments</th><th width="82">Type</th><th>Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>name</td><td>string</td><td><code>true</code></td><td></td><td>The unique name for the new Queue Connection.</td><td></td></tr></tbody></table>

```cfscript
component {

    function configure() {
        newConnection( "default" );
    }

}
```

A `QueueConnectionDefinition` has various methods to configure the Queue Connection.

{% content-ref url="queue-connection.md" %}
[queue-connection.md](queue-connection.md)
{% endcontent-ref %}

### newWorkerPool

This command creates a new `WorkerPoolDefinition` builder component.  It requires a unique `name` that will define this Worker Pool and a `connectionName` that points to an already created Connection.

<table><thead><tr><th>Arguments</th><th>Type</th><th>Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>name</td><td>string</td><td><code>true</code></td><td></td><td>The unique name for the new Worker Pool.</td><td></td></tr><tr><td>connectionName</td><td>string</td><td><code>false</code></td><td></td><td>A reference to an existing Connection.  If not passed in here, the <code>forConnection</code> method must be called to define the Connection.</td><td></td></tr><tr><td>quantity</td><td>numeric</td><td><code>false</code></td><td>1</td><td>The number of workers to spin up for this Worker Pool.</td><td></td></tr><tr><td>queues</td><td>array</td><td><code>false</code></td><td><code>[ * ]</code></td><td>An array of queues that this Worker Pool will work.  A queue of <code>*</code> refers to all queues. Queues will be worked in the order provided.</td><td></td></tr><tr><td>force</td><td>boolean</td><td><code>false</code></td><td><code>false</code></td><td>If <code>false</code>, an exception will be thrown if the Worker Pool name has already been registered.  If <code>true</code>, the new definition will override the existing definition.</td><td></td></tr></tbody></table>

<pre class="language-cfscript"><code class="lang-cfscript"><strong>component {
</strong>
    function configure() {
        newConnection( "default" );
        
        newWorkerPool( "default" ).forConnection( "default" );
    }

}
</code></pre>

A `WorkerPoolDefinition` has various methods to configure the Worker Pool.

{% content-ref url="worker-pool.md" %}
[worker-pool.md](worker-pool.md)
{% endcontent-ref %}

### reset

Removes all Connections and Worker Pools

```cfscript
getInstance( "Config@cbq" ).reset();
```

### Environment Overrides

Just as in your `config/ColdBox.cfc` file or in `ModuleConfig.cfc` files, you can add, remove, or modify Queue Connection or Worker Pool definitions per environment.  You do this by defining a method on your Config component matching the environment name you want to override.

```cfscript
component {

    function configure() {
        newConnection( "default" )
            .provider( "DBProvider@cbq" );
            
        newWorkerPool( "default" )
            .forConnection( "default" )
            .quantity( 3 )
            .timeout( 15 );
    }
    
    /**
     * This method will be called after `configure`
     * and only if the current environment is `development`.
     */
    function development() {
        withConnection( "default" )
            .provider( "SyncProvider@cbq" );
            
        withWorkerPool( "default" )
            .quantity( 1 )
            .timeout( 60 );
    }

}
```

### withConnection

Retrieves an already defined Queue Connection Definition for overriding.  All the same methods are available.

{% content-ref url="queue-connection.md" %}
[queue-connection.md](queue-connection.md)
{% endcontent-ref %}

#### delete

{% hint style="danger" %}
This method is not yet implemented.
{% endhint %}

### withWorkerPool

Retrieves an already defined Worker Pool Definition for overriding.  All the same methods are available.

{% content-ref url="worker-pool.md" %}
[worker-pool.md](worker-pool.md)
{% endcontent-ref %}

#### delete

{% hint style="danger" %}
This method is not yet implemented.
{% endhint %}
