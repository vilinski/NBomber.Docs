# Building blocks

The whole API is mainly built around 3 building blocks: ConnectionPool, Step, Scenario.

```fsharp
// represents pool of connections
// one connection could be Http/WebSockets/SQL or both
type ConnectionPool<'TConnection> = {
    PoolName: string    
    OpenConnection: unit -> 'TConnection
    CloseConnection: ('TConnection -> unit) option
    ConnectionsCount: int
    AliveConnections: 'TConnection[]
}

// represents single executable Step
// it's a basic element which will be executed and measured
type Step =
    | Pull  of PullStep // to model Request-response pattern
    | Push  of PushStep // to model Pub/Sub pattern
    | Pause of TimeSpan // to model pause in your test flow

// represents Scenario which groups steps and execute them sequentially
// on dedicated System.Threading.Task
type Scenario = {
    ScenarioName: string
    TestInit: (unit -> unit) option  // init func will be executed at start of every scenario
    TestClean: (unit -> unit) option // clean func will be executed at end of every scenario
    Steps: Step[]                    // these steps will be executed sequentially, one by one
    Assertions: Assertion[] // list of assertions defined by user
    ConcurrentCopies: int   // specify how many copies of current Scenario to run in parallel    
    Duration: TimeSpan      // execution time of Scenario 
}
```

### Step
Step is a basic element (you can think of Step like a function) which will be executed and measured. 

```fsharp
type Step =
    | Pull  of PullStep // to model Request-response pattern
    | Push  of PushStep // to model Pub/Sub pattern
    | Pause of TimeSpan // to model pause in your test flow
```    

NBomber provides 3 type of steps:
- **Pull** - to model Request-response pattern. You can use Pull step to simulate testing of database, HTTP server and etc.
- **Push** - to model Pub/Sub pattern. You can use Push step to simulate testing of WebSockets, SSE, RabbitMq, Kafka and etc. Usually for testing Pub/Sub you need a trigger and listener. The trigger will trigger some action and then listener will listen on correspond updates.
- **Pause** - to model pause. You can use pause to simulate micro-batching update, or just wait some time on operation complete.

This is how simple Pull step could be defined:

```csharp
// C# example of simple Push step
var reqStep = Step.CreatePull(
    name: "simple step",
    pool: ConnectionPool.None,
    execute: (context) => Task.FromResult(Response.Ok())
);
``` 
```fsharp
// F# example of simple Request step
let reqStep = Step.createPull("simple step", ConnectionPool.none, fun context -> task {
    return Response.Ok() 
})
```

### Scenario
Scenario is basically a container for steps (you can think of Scenario like a Job of sequential operations). It contains optional TestInit and TestClose steps which you can define to prepare or clean test environment (load/restore database, clear cache, clean folders and etc).

```fsharp
// represents Scenario which groups steps and execute them sequentially
// on dedicated System.Threading.Task
type Scenario = {
    ScenarioName: string
    TestInit: (unit -> unit) option  // init func will be executed at start of every scenario
    TestClean: (unit -> unit) option // clean func will be executed at end of every scenario
    Steps: Step[]                    // these steps will be executed sequentially, one by one
    Assertions: Assertion[] // list of assertions defined by user
    ConcurrentCopies: int   // specify how many copies of current Scenario to run in parallel    
    Duration: TimeSpan      // execution time of Scenario 
}
```

> **All steps within one Scenario are executing sequentially**. It helps you model dependently ordered operations. Scenario allows you to specify how many copies will be run in parallel. For example, if you set ConcurrentCopies = 10, it means that 10 copies of the same Scenario will be created and started at the same time.

This is how simple Scenario could be defined and runned:

```csharp
// C# example of Scenario with 2 sequential steps
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
Scenario.create("Sequantial flow", [authUserStep; buyProductStep])
|> Scenario.withConcurrentCopies(10)
|> Scenario.withDuration(TimeSpan.FromSeconds(20.0))
|> NBomberRunner.registerScenario
|> NBomberRunner.runInConsole
```