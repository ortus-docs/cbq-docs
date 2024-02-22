# Queue Connection

A Queue Connection is defined inside your cbq config file using a `QueueConnectionDefinition` builder component.  You can create one of these builder components using the [`newConnection`](./#newconnection) method.

### QueueConnectionDefinition Methods

#### provider

Sets the provider for the Queue Connection.  This can be any valid WireBox mapping and should implement the `IQueueProvider` interface. (There is no need to use the `implements` keyword.)

<table><thead><tr><th>Arguments</th><th width="128">Type</th><th>Required</th><th>Default</th><th>Description</th><th data-hidden><select></select></th></tr></thead><tbody><tr><td>provider</td><td>string</td><td><code>true</code></td><td></td><td>A valid WireBox mapping to the desired Provider.</td><td></td></tr></tbody></table>

#### setProvider

Alias for [`provider`](queue-connection.md#provider).

#### setProperties

Accepts a struct of properties to configure the Queue Connection.  The available properties are usually defined by the Queue Provider being used.

#### setDefaultQueue

Sets the default queue to use for jobs dispatched on this Queue Connection.

#### markAsDefault

Marks this Queue Connection as the default Queue Connection for jobs dispatched without specifying a Queue Connection.

#### setMakeDefault

Alias for [`markAsDefault`](queue-connection.md#markasdefault).
