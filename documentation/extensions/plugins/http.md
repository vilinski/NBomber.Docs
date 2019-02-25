# NBomber.Http

[![Build status](https://ci.appveyor.com/api/projects/status/svujv1049rtf4nb9?svg=true)](https://ci.appveyor.com/project/PragmaticFlowOrg/nbomber-http)
[![NuGet](https://img.shields.io/nuget/v/nbomber.http.svg)](https://www.nuget.org/packages/nbomber.http/)

NBomber plugin for defining HTTP scenarios

### How to install
To install NBomber.Http via NuGet, run this command in NuGet package manager console:
```code
PM> Install-Package NBomber.Http
```

### Contributing
Would you like to help make NBomber even better? We keep a list of issues that are approachable for newcomers under the [good-first-issue](https://github.com/PragmaticFlow/NBomber.Http/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22) label.

### Examples

# [F#](#tab/tabid-1)
```fsharp
open System
open NBomber.FSharp
open NBomber.Http.FSharp

let buildScenario () =

    let step =
        HttpStep.createRequest "GET" "https://www.youtube.com"
        |> HttpStep.withHeader "Accept" "text/html"
        |> HttpStep.withHeader "User-Agent" "Mozilla/5.0"                                         
        |> HttpStep.build "GET request"

        // |> HttpStep.withVersion "2.0"
        // |> HttpStep.withBody(new StringContent ("""{"some":"jsonvalue"}"""))
        // |> HttpStep.withBody(new ByteArrayContent("some byte array"B))

    Scenario.create("test youtube.com", [step])
    |> Scenario.withConcurrentCopies 50
    |> Scenario.withDuration(TimeSpan.FromSeconds 10.0)

[<EntryPoint>]
let main argv =
    buildScenario()
    |> NBomberRunner.registerScenario
    |> NBomberRunner.runInConsole
0
```

# [C#](#tab/tabid-2)
```csharp
using System;
using NBomber.Contracts;
using NBomber.CSharp;
using NBomber.Http.CSharp;

namespace NBomber.Http.Examples.CSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            var scenario = BuildScenario();
            NBomberRunner.RegisterScenarios(scenario)
                         .RunInConsole();
        }

        static Scenario BuildScenario()
        {
            var step = HttpStep.CreateRequest("GET", "https://www.youtube.com")                               
                               .WithHeader("Accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8")
                               .WithHeader("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36")                               
                               .BuildStep("GET request");
                               //.WithVersion("2.0")
                               //.WithBody(new StringContent("{ some json }"))
                               //.WithBody(new ByteArrayContent())

            return ScenarioBuilder.CreateScenario("test youtube.com", step)
                .WithConcurrentCopies(50)
                .WithDuration(TimeSpan.FromSeconds(10));                
        }
    }
}

```
***