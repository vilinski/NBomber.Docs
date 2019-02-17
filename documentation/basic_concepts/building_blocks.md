# Building blocks

The whole API is mainly built around 3 building blocks: Step, Scenario and NBomberRunner. In this page, we just want to present a short definition of basic components and show API usage. In the next pages, you will get a details about all available components and all information about how to model complex load test scenarios. 

```fsharp
// represents single executable Step
// it's a basic element which will be executed and measured
type Step =
    | Action of ActionStep // to model Request-response or Pub/Sub pattern
    | Pause  of TimeSpan   // to model pause in your test flow

// represents Scenario which a container for steps, assertions
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

## API Usage

Step is a basic element of every Scenario which will be executed and measured. You can think of Step like a function which execution time will be measured:
```fsharp
// it's pseudocode example where we measure step's execution time
timer.Start()
step.ExecuteAsync()
timer.Stop()
```

Scenario is basically a container for steps. You can think of Scenario like a job of sequential operations. NBomber is simply running your defined steps in a loop and measure execution time and build statistics on top of it.

```fsharp
// it's pseudocode example which demonstrates 
// how NBomber will execute scenario's steps
while not stop do
    for step in scenario.Steps do
        
        // measure step's execution time
        timer.Start()
        step.Execute()
        timer.Stop()
        
        // append the current execution time to executionTimeList
        executionTimeList.Add(timer.Elapsed.TotalMilliseconds)

// build statistics
let stats = Statistics.apply(executionTimeList)
Reporting.Build(stats)
```

In order to cover any test Scenario, we first need to create a Step. This is how simple Step could be defined:

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
``` 
***

After defining a Step we need to add it to Scenario and then run Scenario via NBomberRunner.

# [F#](#tab/tabid-1)
```fsharp
// creating a step
let pullStep = Step.createAction(...)

// creating scenario and run it via NBomberRunner
Scenario.create("Simple scenario", [pullStep])
|> Scenario.withConcurrentCopies(10)
|> Scenario.withDuration(TimeSpan.FromSeconds(20.0))
|> NBomberRunner.registerScenario
|> NBomberRunner.runInConsole
```

# [C#](#tab/tabid-2)
```csharp
// creating a step
var step = Step.CreateAction(...);

// creating scenario 
var scenario = ScenarioBuilder
    .CreateScenario("Simple scenario", step)
    .WithConcurrentCopies(10)
    .WithDuration(TimeSpan.FromSeconds(20));    

// run scenario via NBomberRunner
NBomberRunner.RegisterScenarios(scenario)             
             .RunInConsole();
```
***