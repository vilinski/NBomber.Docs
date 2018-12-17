# Testing HTTP via Pull Scenario

```csharp
using System;
using System.Net.Http;

using NBomber.Contracts;
using NBomber.CSharp;

namespace CSharp.Example.Http
{
    class Program
    {
        static Scenario BuildScenario()
        {
            HttpRequestMessage CreateHttpRequest()
            {
                var msg = new HttpRequestMessage();
                msg.RequestUri = new Uri("https://github.com/PragmaticFlow/NBomber");
                msg.Headers.TryAddWithoutValidation("Accept", "text/html");                
                return msg;
            }

            var httpPool = ConnectionPool.Create("http pool", () => new HttpClient());

            var step1 = Step.CreatePull("GET html", httpPool, async context =>
            {
                var request = CreateHttpRequest();
                var response = await context.Connection.SendAsync(request);
                return response.IsSuccessStatusCode
                    ? Response.Ok()
                    : Response.Fail(response.StatusCode.ToString());
            });

            return ScenarioBuilder.CreateScenario("test github", step1);
                                  .WithConcurrentCopies(50)
                                  .WithDuration(TimeSpan.FromSeconds(10));
        }

        static void Main(string[] args)
        {            
            var scenario = BuildScenario();
            NBomberRunner.RegisterScenarios(scenario)
                         .RunInConsole();
        }
    }
}

```