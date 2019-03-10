# Step

Step is a basic element of every scenario which will be executed and measured. Every step is running in separated Task.

```fsharp
type Step =
    | Action of ActionStep // to model Request-response or Pub/Sub pattern
    | Pause  of TimeSpan   // to model pause in your test flow
```

NBomber provides 2 types of step:
- **Action** - You can use it to simulate any possible **Pull/Push request** for testing any system: database, HTTP/WebSockets server, any message broker like RabbitMQ, Kafka, etc.
- **Pause** - You can use pause to simulate micro-batching update, or just wait a certain period between sequential operations.

You can think of Step like a function which execution time will be measured:
```fsharp
// it's pseudocode example where we measure step's execution time
let start = getCurrentTime()
execFunc()
let finish = getCurrentTime()

let latency = finish - start
```

## API

# [F#](#tab/tabid-1)
```fsharp
module Step =
    let createAction (name: string,
                      pool: IConnectionPool<'TConnection>,
                      execute: StepContext<'TConnection> -> Task<Response>): IStep

    let createPause (duration: TimeSpan): IStep
```

# [C#](#tab/tabid-2)
```csharp
public class Step
{
    public static IStep CreateAction<TConnection>(
        string name,
        IConnectionPool<TConnection> pool,
        Func<StepContext<TConnection>, Task<Response>> execute);

    public static IStep CreatePause(TimeSpan duration);
}
```
***

## API Usage

# [F#](#tab/tabid-1)
```fsharp
// example of simple Pull step
let pullStep = Step.createAction("pull step", ConnectionPool.none, fun context -> task {

    // you can do any logic here: go to http, websocket etc
    // for example, you can send http request and wait on response
    let! response = httpClient.SendAsync(request)

    // or query MongoDb and wait on response
    let find = mongoCollection.Find(fun u -> u.IsActive == true)
    let! data = find.ToListAsync()

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
var pullStep = Step.CreateAction("pull step", ConnectionPool.None, async (context) =>
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
var pushStep = Step.CreateAction("push step", ConnectionPool.None, async (context) =>
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

Every step is running in separated Task and has its own context. It's a very useful component which contains all related step's context information. NBomber runtime will automatically create a StepContext and attach it to running Step so you don't need to worry about creation you just need know how to work with StepContext.

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
***

### Payload
You can use Payload to model dependently ordered operations like: login() -> openProduct() -> buyProduct() -> logout(). Basically, it gives you a way to propagate result from step1 to step2.

# [F#](#tab/tabid-1)
```fsharp
// every step has a way to pass result of operation to the next step
// let's assume we are in step 1 and we want to return
// the result of 2 + 2 operation to the next step
let step1 = Step.createAction("step 1", ConnectionPool.none, fun context -> task {
    return Response.Ok(2 + 2)
})

// in step 2 we can take the value of Response.Ok(2 + 2) via context.Payload
let step2 = Step.createAction("step 2", ConnectionPool.none, fun context -> task {
    printf "%A" context.Payload // will print 4
    return Response.Ok()
})
```

# [C#](#tab/tabid-2)
```csharp
// every step has a way to pass result of operation to the next step
// let's assume we are in step 1 and we want to return
// the result of 2 + 2 operation to the next step
var step1 = Step.CreateAction("step 1", ConnectionPool.None, async (context) =>
{
    return Response.Ok(2 + 2);
});

// in step 2 we can take the value of Response.Ok(2 + 2) via context.Payload
var step2 = Step.CreateAction("step 1", ConnectionPool.None, async (context) =>
{
    Console.WriteLine(context.Payload); // will print 4
    return Response.Ok();
});
```
***

### CorrelationId
Every step has CorrelationId which will be automatically created by NBomber. You can use StepContext.CorrelationId to model scenarios where you need to observe specific message type tagged by your id. All sequential steps [step1; step2; step3] will have the same CorrelationId within one Task.

# [F#](#tab/tabid-1)
```fsharp
// the function which NBomber use to create CorrelationId for every copy of Step
let createCorrelationId (scnName: string, concurrentCopies: int) =
    [|0 .. concurrentCopies - 1|]
    |> Array.map(fun i -> sprintf "%s_%i" scnName i)
```

# [C#](#tab/tabid-2)
```csharp
// the function which NBomber use to create CorrelationId for every copy of Step
public string[] CreateCorrelationId(string scnName, int concurrentCopies)
{
    Enumerable.Range(0, concurrentCopies - 1)
        .Select(i => $"{scnName}_{i}")
        .ToArray();
}
```
***

### Connection and ConnectionPool
Connection represents one connection from the ConnectionPool which is attached to the step. ConnectionPool is a component which represents a given pool of connections. Usually, you should use ConnectionPool to model dedicated connection per Step. For example, you wanna create a Step where you want to send a message via WebSockets to some server. In this case, you interested to do not share one global WebSocket connection between all concurrent copies of your Step. Instead, you would like that every copy of Step has its own WebSocket connection.

# [F#](#tab/tabid-1)
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
let step = Step.createAction("step", webSocketsPool, fun context -> task {
    do! context.Connection.SendAsync("ping")
    return Response.Ok()
})
```

# [C#](#tab/tabid-2)
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
var step = Step.CreateAction("step", webSocketsPool, async (context) =>
{
    await context.Connection.SendAsync("ping");
    return Response.Ok();
})
```
***

### CancellationToken
CancellationToken is a standard mechanics for canceling long-running operations. NBomber relay on it when it stops scenario. The good practice is to design your steps to be cancellable.

# [F#](#tab/tabid-1)
```fsharp
// in this example, we use CancellationToken
// to allow NBomber cancel running Step
let httpClient = HttpClient()

let step = Step.createAction("step", ConnectionPool.none, fun context -> task {
    let! response = httpClient.SendAsync(createHttpRequest(), context.CancellationToken)
    return Response.Ok(response)
})
```

# [C#](#tab/tabid-2)
```csharp
// in this example, we use CancellationToken
// to allow NBomber cancel running Step
var httpClient = new HttpClient();

var step = Step.CreateAction("step", ConnectionPool.None, async (context) =>
{
    var response = await httpClient.SendAsync(createHttpRequest(), context.CancellationToken);
    return Response.Ok(response);
})
```
***
