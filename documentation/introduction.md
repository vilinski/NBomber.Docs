# Introduction

### Step 1. Installation
To install NBomber via NuGet, run this command in NuGet package manager console:
```code
PM> Install-Package NBomber
```

### Step 2. Design and run a load test scenario
# [C#](#tab/tabid-1)
```csharp
using System;
using System.Threading.Tasks;
using NBomber.Contracts;
using NBomber.CSharp;

namespace Examples
{
    class Program
    {
        static void Main(string[] args)
        {   
            // first, you need to create a step
            var step = Step.Create("step", async context =>
            {
                // you can do any logic here: go to http, websocket etc
                await Task.Delay(TimeSpan.FromSeconds(0.1));
                return Response.Ok();
            });

            // after creating a step you should add it to Scenario.
            var scenario = ScenarioBuilder.CreateScenario("Hello World!", step);

            NBomberRunner.RegisterScenarios(scenario)
                         .RunInConsole();
        }
    }
}             
```

# [F#](#tab/tabid-2)
```fsharp
open System
open System.Threading.Tasks
open FSharp.Control.Tasks.V2.ContextInsensitive
open NBomber.Contracts
open NBomber.FSharp

[<EntryPoint>]
let main argv =

    // first, you need to create a step
    let step = Step.create("simple step", fun context -> task {        
        // you can do any logic here: go to http, websocket etc        
        do! Task.Delay(TimeSpan.FromSeconds(0.1))
        return Response.Ok() 
    })
        
    let scenario = Scenario.create "Hello World!" [step]

    NBomberRunner.registerScenarios [scenario]
    |> NBomberRunner.runInConsole
```
***

### Step 3. View results
View the results. Here is an example of console output from the above benchmark:
```
 Scenario: Hello World from NBomber!, execution time: 00:00:10
+-----------+---------------+-----+--------+-----+-----+------+-----+-----+-----+-----+
| step name | request_count | OK  | failed | RPS | min | mean | max | 50% | 75% | 95% |
+-----------+---------------+-----+--------+-----+-----+------+-----+-----+-----+-----+
| pull step | 923           | 923 | 0      | 92  | 99  | 108  | 126 | 109 | 111 | 114 |
+-----------+---------------+-----+--------+-----+-----+------+-----+-----+-----+-----+
```

### Step 4. Analyze results
In your bin directory, you can find a folder 'results' with *.txt, *.html, *.csv, *.md reports.

### Step 5. Integrate load test in your CI/CD pipline
If you decided to add load test in your CI/CD pipline NBomber provides an integration with:
- [NUnit](https://github.com/PragmaticFlow/NBomber/blob/dev/examples/CSharp/CSharp.Examples.NUnit/Tests.cs)
- [XUnit](https://github.com/PragmaticFlow/NBomber/blob/dev/examples/FSharp/FSharp.Examples.XUnit/Tests.fs) 
- [Expecto](https://github.com/PragmaticFlow/NBomber/blob/dev/examples/FSharp/FSharp.Examples.Expecto/Tests.fs) 

### Step 6. Sink your test results in any data storage to track performance trends
If you decided to track your performance trends and analyze historical data NBomber provides an integrations with:
- [InfluxDB](https://github.com/PragmaticFlow/NBomber.Sinks.InfluxDB/blob/master/examples/NBomber.Sinks.InfluxDB.Examples.CSharp/Program.cs)

### Next steps
NBomber provides a lot of features which help to load test any system. If you want to know more about NBomber features, checkout the Overview page. If you have any questions, checkout the FAQ page. If you didn't find answer for your question on this page, ask it on gitter or create an issue.