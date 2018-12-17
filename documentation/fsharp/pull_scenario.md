# Testing HTTP via Pull Scenario

```fsharp
open System
open System.Net.Http

open FSharp.Control.Tasks.V2.ContextInsensitive

open NBomber.Contracts
open NBomber.FSharp

let buildScenario () =

    let createHttpRequest () =
        let msg = new HttpRequestMessage()
        msg.RequestUri <- Uri("https://github.com/PragmaticFlow/NBomber")
        msg.Headers.TryAddWithoutValidation("Accept", "text/html") |> ignore        
        msg
    
    let pool = ConnectionPool.create("httpPool", fun () -> new HttpClient())    

    let step1 = Step.createPull("GET html", pool, fun context -> task {        
        let! response = createHttpRequest() |> context.Connection.SendAsync
        return if response.IsSuccessStatusCode then Response.Ok()
               else Response.Fail(response.StatusCode.ToString()) 
    })

    Scenario.create("test github", [step1])
    |> Scenario.withConcurrentCopies(50)
    |> Scenario.withDuration(TimeSpan.FromSeconds(10.0))

[<EntryPoint>]
let main argv =    
    
    buildScenario()
    |> NBomberRunner.registerScenario
    |> NBomberRunner.runInConsole

    0 // return an integer exit code
```