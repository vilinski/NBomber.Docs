# Step

Step is a basic element of every scenario which will be executed and measured. Every step is running in separated thread. NBomber is using lightweight threads via System.Threading.Tasks.

```fsharp
type Step =
    | Action of ActionStep // to model Request-response or Pub/Sub pattern
    | Pause  of TimeSpan   // to model pause in your test flow
```

You can think of Step like a function which execution time will be measured:
```fsharp
// it's pseudocode example where we measure step's execution time
timer.Start()
step.ExecuteAsync()
timer.Stop()
```

NBomber provides 2 types of step:
- **Action** - You can use it to simulate any possible **Pull/Push request** for testing any system: database, HTTP/WebSockets server, any message broker like RabbitMQ, Kafka, etc.
- **Pause** - You can use pause to simulate micro-batching update, or just wait a certain period between sequential operations.

This is how simple step could be defined:

# [F#](#tab/tabid-1)
```fsharp
// example of simple Pull step
let pullStep = Step.createAction("pull step", ConnectionPool.none, fun context -> task {
    
    // you can do any logic here: go to http, websocket etc    
    // for example, you can send http request and wait on response
    let! response = httpClient.SendAsync(request)
    
    // or query MongoDb and wait on response
    let! data = mongoCollection.Find(u => u.IsActive == true).ToListAsync()

    // in case of error, you can return Response.Fail()     
    return Response.Ok()    
})

// example of simple Push step
let pushStep = Step.createAction("push step", ConnectionPool.none, fun context -> task {
        
    // you can do any logic here:
    // for example, you can wait on response from websockets    
    let! response = webSocketClient.ReceiveMessageAsync()
    
    // or wait on response from rabbitMQ broker
    let! data = rabbitMQ.ReceiveUpdateAsync()
        
    return Response.Ok() 
})

// example of Pause step
let pause_2_sec = Step.createPause(TimeSpan.FromSeconds(2.0))
```

# [C#](#tab/tabid-2)
```csharp
// example of simple Pull step
var pullStep = Step.CreateAction("pull step", ConnectionPool.None, execute: async (context) => 
{
    // you can do any logic here: go to http, websocket etc        
    // for example, you can send http request and wait on response
    var response = await httpClient.SendAsync(request);

    // or query MongoDb and wait on response
    var data = await mongoCollection.Find(u => u.IsActive == true).ToListAsync();        

    // in case of error, you can return Response.Fail()
    return Response.Ok();
});

// example of simple Push step
var pushStep = Step.CreateAction("push step", ConnectionPool.None, execute: async (context) => 
{
    // you can do any logic here:
    // for example, you can wait on response from websockets    
    var response = await webSocketClient.ReceiveMessageAsync();
    
    // or wait on response from rabbitMQ broker
    var data = await rabbitMQ.ReceiveUpdateAsync();
        
    return Response.Ok(); 
});

// example of Pause step
var pause_2_sec = Step.CreatePause(TimeSpan.FromSeconds(2));
``` 
***

## Step context

Every step is running in separated Task<TResponse> and has its own context. It's a very useful component which contains all related step's information. You can use StepContext to model scenarios which has sequential operations like: loginUser -> makeOrder -> logoutUser. NBomber runtime will automatically create a StepContext and attach it to running Step so you don't need to warry about creation you just need know how to work with StepContext.

# [F#](#tab/tabid-1)
```fsharp
type StepContext<'TConnection> = {
    CorrelationId: string
    CancellationToken: CancellationToken
    Connection: 'TConnection
    Payload: obj
}
```

# [C#](#tab/tabid-2)
```csharp
public sealed class StepContext<TConnection>
{
    public string CorrelationId { get; }
    public CancellationToken CancellationToken { get; }
    public TConnection Connection { get; }
    public object Payload { get; }
}
```