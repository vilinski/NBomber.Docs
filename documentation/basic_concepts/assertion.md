# Assertion

NBomber provides a way to specify assertion for concreate step within a scenario.

# [F#](#tab/tabid-1)
```fsharp
type Assertion =
    static member forStep (stepName: string, assertion: Statistics -> bool, ?label: string)
```

# [C#](#tab/tabid-2)
```csharp
public class Assertion
{
    public static IAssertion ForStep(string stepName, Func<Statistics, bool> assertion, string label = null);
}
```

#### API usage

# [F#](#tab/tabid-1)
```fsharp
module Tests

open System
open System.Threading.Tasks

open Xunit
open FSharp.Control.Tasks.V2.ContextInsensitive

open NBomber.Contracts
open NBomber.FSharp

let buildScenario () =

    let step1 = Step.createAction("simple step", ConnectionPool.none, fun context -> task {
        do! Task.Delay(TimeSpan.FromSeconds(0.1))
        return Response.Ok(sizeBytes = 1024)
    })

    Scenario.create("xunit hello world", [step1])    

[<Fact>]
let ``XUnit test`` () =    
    
    let assertions = [              
       Assertion.forStep("simple step", (fun stats -> stats.RPS > 8), "RPS > 8")              
       Assertion.forStep("simple step", (fun stats -> stats.AllDataMB >= 0.01), "AllDataMB >= 0.01")
    ]

    buildScenario()
    |> Scenario.withConcurrentCopies(1)
    |> Scenario.withWarmUpDuration(TimeSpan.Zero)
    |> Scenario.withDuration(TimeSpan.FromSeconds(2.0))
    |> Scenario.withAssertions(assertions)
    |> NBomberRunner.registerScenario
    |> NBomberRunner.runTest
```

# [C#](#tab/tabid-2)
```csharp
using System;
using System.Threading.Tasks;

using NUnit.Framework;

using NBomber.Contracts;
using NBomber.CSharp;

namespace CSharp.Examples.NUnit
{
    public class Tests
    {
        Scenario BuildScenario()
        {   
            var step1 = Step.CreateAction("simple step", ConnectionPool.None, async context =>
            {
                await Task.Delay(TimeSpan.FromSeconds(0.1));
                return Response.Ok(sizeBytes: 1024);
            });

            return ScenarioBuilder.CreateScenario("nunit hello world", step1);                
        }

        [Test]
        public void Test()
        {
            var assertions = new[] {                              
               Assertion.ForStep("simple step", stats => stats.RPS > 8, "RPS > 8"),               
               Assertion.ForStep("simple step", stats => stats.AllDataMB >= 0.01, "AllDataMB >= 0.01")
            };

            var scenario = BuildScenario()
                .WithConcurrentCopies(1)
                .WithWarmUpDuration(TimeSpan.Zero)
                .WithDuration(TimeSpan.FromSeconds(2))
                .WithAssertions(assertions);

            NBomberRunner.RegisterScenarios(scenario)
                         .RunTest();            
        }
    }
}
```
