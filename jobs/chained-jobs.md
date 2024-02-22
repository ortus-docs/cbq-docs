# Chained Jobs

Job chains mean that after a Job completes successfully it dispatches the next Job in the chain.  If a Job fails, no more Jobs in the chain are dispatched.

## Dispatching Jobs from another Job

One way to make a Job chain is to dispatch a Job from inside another Job. This gives you consistency in your Job chain, and a logical code path to follow.

```cfscript
// FulfillOrderJob.cfc
component extends="cbq.models.Jobs.AbstractJob" {

    function handle() {
        var productId = getProperties().productId;

      	processPayment( productId );

        getInstance( "SendProductLinkEmail" )
            .onConnection( "fulfillment" )
            .setProperties( {
                "productId" : productId,
                "userId" : getProperties().userId
            } )
            .dispatch();
    }

}
```

## Creating a Chain

Another way to make a Job chain is to create is when dispatching the Job.  This gives you flexibility to compose Job chains at runtime.

```cfscript
// handlers/Main.cfc
component {
  
    property name="cbq" inject="cbq@cbq";

    function create() {
        cbq.chain( [
            cbq.job( "FulfillOrderJob", { "productId": rc.productId } ),
            cbq.job( job = "SendProductLinkEmail", properties = {
              "productId": rc.productId,
              "userId": auth().getUserId()
            }, connection = "fulfillment" ),
        ] ).dispatch();
    }

}
```

## Dispatching Jobs on Failure

Utilizing the `onFailure` method of a Job, we can dispatch another Job if our Job fails.

```cfscript
// FulfillOrderJob.cfc
component extends="cbq.models.Jobs.AbstractJob" {

    function handle() {
        // ...
    }
    
    function onFailure( e ) {
        getInstance( "SendOrderProblemEmail" )
            .onConnection( "fulfillment" )
            .setProperties( {
                "productId" : productId,
                "userId" : getProperties().userId,
                "message" : e.message
            } )
            .dispatch();
    }

}
```
