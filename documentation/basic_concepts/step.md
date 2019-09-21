# Step

Step is a basic element of every scenario which will be executed and measured. Every step is running in separated Task.

```fsharp
type Step = {
    StepName: string    
    Execute: StepContext -> Task<Response>
    ConnectionPool: ConnectionPool
}
```

You should use Step to simulate any possible **Pull/Push request** for testing any database, HTTP/WebSockets server, any message broker like RabbitMQ, Kafka, etc. Also with Step you can simulate pause(to simulate micro-batching update, or just wait a certain period between sequential operations).

You can think of Step like a function which execution time will be measured:
```typescript
// it's pseudocode example where we measure step's execution time
var start = getCurrentTime()
    execFunc()
var end = getCurrentTime()

var latency = end - start
```

## API

# [C#](#tab/tabid-1)
```csharp
public class Step
{
    public static IStep Create<TConnection>(
        string name,         
        Func<StepContext<TConnection>, Task<Response>> execute);
}
```

# [F#](#tab/tabid-2)
```fsharp
module Step =
    let create (name: string,                 
                execute: StepContext<'TConnection> -> Task<Response>): IStep
```
***

## API Usage

# [C#](#tab/tabid-1)
```csharp
// example of simple Pull step
var pullStep = Step.Create("pull step", async (context) => 
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
var pushStep = Step.Create("push step", async (context) => 
{
    // you can do any logic here:
    // for example, you can wait on response from websockets    
    var response = await webSocketClient.ReceiveMessageAsync();
    
    // or wait on response from rabbitMQ broker
    var data = await rabbitMQ.ReceiveUpdateAsync();
        
    return Response.Ok(); 
});
```

# [F#](#tab/tabid-2)
```fsharp
// example of simple Pull step
let pullStep = Step.create("pull step", fun context -> task {
    
    // you can do any logic here: go to http, websocket etc    
    // for example, you can send http request and wait on response
    let! response = httpClient.SendAsync(request)
    
    // or query MongoDb and wait on response
    let! data = mongoCollection.Find(u => u.IsActive == true).ToListAsync()

    // in case of error, you can return Response.Fail()     
    return Response.Ok()    
})

// example of simple Push step
let pushStep = Step.create("push step", fun context -> task {
        
    // you can do any logic here:
    // for example, you can wait on response from websockets    
    let! response = webSocketClient.ReceiveMessageAsync()
    
    // or wait on response from rabbitMQ broker
    let! data = rabbitMQ.ReceiveUpdateAsync()
        
    return Response.Ok() 
})
```
***

## StepContext

Every step is running in separated Task and has its own context. It's a very useful component which contains all related step's context information. NBomber runtime will automatically create a StepContext and attach it to running Step so you don't need to worry about creation you just need know how to work with StepContext.

# [C#](#tab/tabid-1)
```csharp
public sealed class StepContext<TConnection>
{
    public string CorrelationId { get; }
    public CancellationToken CancellationToken { get; }
    public TConnection Connection { get; }
    public object Data { get; }
}
```

# [F#](#tab/tabid-2)
```fsharp
type StepContext<'TConnection> = {
    CorrelationId: string
    CancellationToken: CancellationToken
    Connection: 'TConnection
    Data: obj
}
```
***

### StepContext.Data
You can use Data to model dependently ordered operations like: login() -> openProduct() -> buyProduct() -> logout(). Basically, it gives you a way to propagate result from step1 to step2.

# [C#](#tab/tabid-1)
```csharp
// every step has a way to pass result of operation to the next step
// let's assume we are in step 1 and we want to return 
// the result of 2 + 2 operation to the next step
var step1 = Step.Create("step 1", async (context) => 
{
    return Response.Ok(2 + 2);
});

// in step 2 we can take the value of Response.Ok(2 + 2) via context.Data
var step2 = Step.Create("step 1", async (context) => 
{
    Console.WriteLine(context.Data); // will print 4    
    return Response.Ok();
});
```

# [F#](#tab/tabid-2)
```fsharp
// every step has a way to pass result of operation to the next step
// let's assume we are in step 1 and we want to return 
// the result of 2 + 2 operation to the next step
let step1 = Step.create("step 1", fun context -> task { 
    return Response.Ok(2 + 2)
})

// in step 2 we can take the value of Response.Ok(2 + 2) via context.Data
let step2 = Step.create("step 2", fun context -> task { 
    printf "%A" context.Data // will print 4
    return Response.Ok()
})
```
***

### StepContext.CorrelationId
Every step has CorrelationId which will be automatically created by NBomber. You can use StepContext.CorrelationId to model scenarios where you need to observe specific message type tagged by your id. All sequential steps [step1; step2; step3] will have the same CorrelationId within one Task.

# [C#](#tab/tabid-1)
```csharp
// the function which NBomber use to create CorrelationId for every copy of Step
public string[] CreateCorrelationId(string scnName, int concurrentCopies)
{
    Enumerable.Range(0, concurrentCopies - 1)
        .Select(i => $"{scnName}_{i}")
        .ToArray();
}
```

# [F#](#tab/tabid-2)
```fsharp
// the function which NBomber use to create CorrelationId for every copy of Step
let createCorrelationId (scnName: string, concurrentCopies: int) =
    [|0 .. concurrentCopies - 1|] 
    |> Array.map(fun i -> sprintf "%s_%i" scnName i)
```
***

### StepContext.Connection and ConnectionPool
Connection represents one connection from the ConnectionPool which is attached to the step. ConnectionPool is a component which represents a given pool of connections. Usually, you should use ConnectionPool to model dedicated connection per Step. For example, you wanna create a Step where you want to send a message via WebSockets to some server. In this case, you interested to do not share one global WebSocket connection between all concurrent copies of your Step. Instead, you would like that every copy of Step has its own WebSocket connection.

# [C#](#tab/tabid-1)
```csharp
// in order to create an one connection per step
// we need to create a ConnectionPool and then attach it to the Step
var webSocketsPool = ConnectionPool.Create(
    name: "webSockets pool", 
    
    openConnection: async (token) => 
    {
        var ws = new ClientWebSocket();
        await ws.ConnectAsync(new Uri("ws://localhost:55555"), token);
        return ws;
    },
    
    closeConnection: async (connection, token) => 
    {
        return await connection.CloseAsync(WebSocketCloseStatus.NormalClosure, token);
    }
);

// here, we attach webSocketsPool to the Step
// and use context.Connection to get access to dedicated connection
var step = Step.Create("step", async (context) =>
{ 
    await context.Connection.SendAsync("ping");
    return Response.Ok();
}, 
webSocketsPool)
```

# [F#](#tab/tabid-2)
```fsharp
// in order to create an one connection per step
// we need to create a ConnectionPool and then attach it to the Step
let webSocketsPool = 
    ConnectionPool.create(
        "webSockets pool", 
        
        openConnection = fun (token) -> task {
            let ws = ClientWebSocket()
            do! ws.ConnectAsync(Uri("ws://localhost:55555"), token)
            return ws
        },
        
        closeConnection = fun (connection, token) -> task {
            do! connection.CloseAsync(token)
        }
    )

// here we attach webSocketsPool to the Step
// and use context.Connection to get access to dedicated connection
let step = Step.create("step", webSocketsPool, fun context -> task { 
    do! context.Connection.SendAsync("ping")
    return Response.Ok()
})
```
***

### StepContext.CancellationToken
CancellationToken is a standard mechanics for canceling long-running operations. NBomber relay on it when it stops scenario. The good practice is to design your steps to be cancellable.

# [C#](#tab/tabid-1)
```csharp
// in this example, we use CancellationToken 
// to allow NBomber cancel running Step
var httpClient = new HttpClient();

var step = Step.Create("step", async (context) =>
{ 
    var response = await httpClient.SendAsync(createHttpRequest(), context.CancellationToken);
    return Response.Ok(response);
})
```

# [F#](#tab/tabid-2)
```fsharp
// in this example, we use CancellationToken 
// to allow NBomber cancel running Step
let httpClient = HttpClient()

let step = Step.create("step", fun context -> task { 
    let! response = httpClient.SendAsync(createHttpRequest(), context.CancellationToken)
    return Response.Ok(response)
})
```
***