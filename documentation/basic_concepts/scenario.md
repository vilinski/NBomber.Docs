# Scenario

Scenario is basically a container for steps (you can think of Scenario like a Job of sequential operations). It contains optional TestInit and TestClose functions which you can define to prepare or clean test environment (load/restore database, clear cache, clean folders and etc).

```fsharp
// represents Scenario which groups steps and execute them sequentially
// on dedicated System.Threading.Task
type Scenario = {
    ScenarioName: string
    TestInit: (ScenarioContext -> Task) option  // init scenario's resources
    TestClean: (ScenarioContext -> Task) option // clean scenario's resources
    Steps: Step[]            // these steps will be executed sequentially, one by one
    Assertions: Assertion[]  // list of assertions defined by user
    ConcurrentCopies: int    // specify how many copies of current Scenario to run in parallel    
    WarmUpDuration: TimeSpan // execution time of warm-up before start bombing 
    Duration: TimeSpan       // execution time of Scenario 
}
```

> [!NOTE]
> **All steps within one Scenario are executing sequentially**. It helps you model dependently ordered operations. Scenario allows you to specify how many copies of the same Scenario will be run in parallel. For example, if you set ConcurrentCopies = 10, it means that 10 copies of the same Scenario will be created and started at the same time on 10 different Tasks.

This is how simple Scenario could be defined and runned:

# [C#](#tab/tabid-1)
```csharp
// creating a step
var step = Step.Create(...);

// creating scenario 
var scenario = ScenarioBuilder
    .CreateScenario("Simple scenario", step)
    .WithConcurrentCopies(10)
    .WithDuration(TimeSpan.FromSeconds(20));    

// run scenario via NBomberRunner
NBomberRunner.RegisterScenarios(scenario)             
             .RunInConsole();
```

# [F#](#tab/tabid-2)
```fsharp
let pullStep = Step.create(...)

// creating scenario and run it via NBomberRunner
Scenario.create "Simple scenario" [pullStep]
|> Scenario.withConcurrentCopies(10)
|> Scenario.withDuration(TimeSpan.FromSeconds 20.0)
|> NBomberRunner.registerScenario
|> NBomberRunner.runInConsole
```
***

## Running different scenarios in parallel

**NBomber allows you to register and run several different scenarios in parallel.** For example, in one scenario you can insert/send data, in another one you will read/query data. For this you need to define a few scenarios and register them, that's it.

# [C#](#tab/tabid-1)
```csharp
// these scenarios will be run in parallel             
var insertScenario = ScenarioBuilder.CreateScenario("insert mongo", insertStep);
var readScenario = ScenarioBuilder.CreateScenario("read mongo", readStep);

NBomberRunner.RegisterScenarios(insertScenario, readScenario) 
             .RunInConsole();
```

# [F#](#tab/tabid-2)
```fsharp
// these scenarios will be run in parallel             
let insertScenario = Scenario.create "insert mongo" [insertStep]
let readScenario = Scenario.create "read mongo" [readStep]

[insertScenario; readScenario]
|> NBomberRunner.registerScenarios
|> NBomberRunner.runInConsole
```
***