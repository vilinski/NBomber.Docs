# Building blocks

The whole API is mainly built around 2 building blocks: Step and Scenario.

```fsharp
// represents single executable Step
// it's a basic element which will be executed and measured
type Step =
    | Action of ActionStep // to model Request-response or Pub/Sub pattern
    | Pause  of TimeSpan   // to model pause in your test flow

// represents Scenario which groups steps and execute them sequentially
// on dedicated System.Threading.Task
type Scenario = {
    ScenarioName: string
    TestInit: (unit -> Task) option  // init func will be executed at start of every scenario
    TestClean: (unit -> Task) option // clean func will be executed at end of every scenario
    Steps: Step[]                    // these steps will be executed sequentially, one by one
    Assertions: Assertion[]  // list of assertions defined by user
    ConcurrentCopies: int    // specify how many copies of current Scenario to run in parallel    
    WarmUpDuration: TimeSpan // execution time of warm-up before start bombing 
    Duration: TimeSpan       // execution time of Scenario 
}
```

### Step
Step is a basic element (you can think of Step like a function) which will be executed and measured. 

```fsharp
type Step =
    | Action of ActionStep // to model Request-response or Pub/Sub pattern
    | Pause  of TimeSpan   // to model pause in your test flow
```    

NBomber provides 2 type of steps:
- **Action** - You can simulate any possible **Pull requests** for testing database, HTTP server, etc. Also, you will be capable to simulate **Push requests** like waiting on notification from WebSockets or from any message broker like RabbitMQ, Kafka, etc.
- **Pause** - You can use pause to simulate micro-batching update, or just wait a certain period between sequential operations.

This is how simple step could be defined:

```csharp
// C# example of simple step
var step = Step.CreateAction(name: "simple step", pool: ConnectionPool.None, execute: async (context) => 
{
    // you can do any logic here: go to http, websocket etc        
    // for example, you can send http request and wait on response
    var response = await httpClient.SendAsync(request);

    // or query MongoDb and wait on response
    var data = await mongoCollection.Find(u => u.IsActive == true).ToListAsync();        

    return Response.Ok();
});
``` 
```fsharp
// F# example of simple pull step
let step = Step.createAction("simple step", ConnectionPool.none, fun context -> task {
    
    // you can do any logic here: go to http, websocket etc    
    // for example, you can send http request and wait on response
    let! response = httpClient.SendAsync(request)
    
    // or query MongoDb and wait on response
    let! data = mongoCollection.Find(u => u.IsActive == true).ToListAsync()
        
    return Response.Ok() 
})
```

### Scenario
Scenario is basically a container for steps (you can think of Scenario like a Job of sequential operations). It contains optional TestInit and TestClose functions which you can define to prepare or clean test environment (load/restore database, clear cache, clean folders and etc).

```fsharp
// represents Scenario which groups steps and execute them sequentially
// on dedicated System.Threading.Task
type Scenario = {
    ScenarioName: string
    TestInit: (unit -> Task) option  // init func will be executed at start of every scenario
    TestClean: (unit -> Task) option // clean func will be executed at end of every scenario
    Steps: Step[]                    // these steps will be executed sequentially, one by one
    Assertions: Assertion[]  // list of assertions defined by user
    ConcurrentCopies: int    // specify how many copies of current Scenario to run in parallel    
    WarmUpDuration: TimeSpan // execution time of warm-up before start bombing 
    Duration: TimeSpan       // execution time of Scenario 
}
```

> **All steps within one Scenario are executing sequentially**. It helps you model dependently ordered operations. Scenario allows you to specify how many copies will be run in parallel. For example, if you set ConcurrentCopies = 10, it means that 10 copies of the same Scenario will be created and started at the same time.

This is how simple Scenario could be defined and runned:

```csharp
// C# example of Scenario with 2 sequential steps
// creating two sequential steps
var authUserStep   = Step.CreateAction(...);
var buyProductStep = Step.CreateAction(...);

var scenario = ScenarioBuilder.CreateScenario("Sequantial flow", authUserStep, buyProductStep)
                              .WithConcurrentCopies(10)
                              .WithDuration(TimeSpan.FromSeconds(20));    

// NBomberRunner allows you to register and run several scenarios in parallel
// for example on Scenario will contain steps to read from a database
// and another one will contain steps to insert data to the database
NBomberRunner.RegisterScenarios(scenario)             
             .RunInConsole();
```
```fsharp
// F# example of Scenario with 2 sequential steps
// creating two sequential steps
let authUserStep   = Step.createAction(...)
let buyProductStep = Step.createAction(...)

Scenario.create("Sequantial flow", [authUserStep; buyProductStep])
|> Scenario.withConcurrentCopies(10)
|> Scenario.withDuration(TimeSpan.FromSeconds(20.0))
|> NBomberRunner.registerScenario
|> NBomberRunner.runInConsole
```
