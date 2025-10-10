# Walkthrough

## Install cbq

Refer to the [Installation](installation.md) section for general installation instructions as well as instructions for your specific [provider](../configuration/providers/).

## Create a Queue Connection

Open up your `config/cbq.cfc` file and create your first Queue Connection:

```cfscript
component {

    function configure() {
        newConnection( "default" )
            .setProvider( "ColdBoxAsyncProvider@cbq" );
    }

}
```

## Create a Worker Pool

Next, create a Worker Pool for your new Queue Connection.  This allows our application to work the Jobs we will dispatch.

```cfscript
component {

    function configure() {
        newConnection( "default" )
            .setProvider( "ColdBoxAsyncProvider@cbq" );
            
        newWorkerPool( "default" )
            .forConnection( "default" );
    }

}
```

## Define your first Job

A Job is a CFC that extends `cbq.models.Jobs.AbstractJob`.  It can live anywhere in your application.

```cfscript
// models/jobs/emails/SendWelcomeEmailJob.cfc
component extends="cbq.models.Jobs.AbstractJob" {
  
    property name="userId";

    function handle() {
        log.info( "Sending a Welcome email to User ###getUserId()#" );
    
        /* sample code
      	var user = getInstance( "User" ).findOrFail( getUserId() );
      
        getInstance( "MailService@cbmailservices" )
            .newMail(
                from = "no-reply@example.com",
                to = user.getEmail(),
                subject = "Welcome!",
                type = "html"
            )
            .setView( "/_emails/users/welcome" )
            .setBodyTokens( {
                "firstName" : user.getFirstName(),
                "lastName" : user.getLastName()
            } )
            .send();
        */
    }

}
```

## Create an instance of your Job

You can create an instance of your Job anywhere in your code — handlers, services, models, etc. Populate it with the specific data needed for this instance.

```cfscript
var job = getInstance( "SendWelcomeEmailJob" );
job.setUserId( newUser.getId() );
```

## Dispatch your Job

Once your Job is created and configured, `dispatch` it to the Queue Connection.

```cfscript
job.dispatch();
```

## Watch your job get executed

Check out LogBox to see your Job being executed.  Congratulations! You've dispatched your first background Job using cbq!
