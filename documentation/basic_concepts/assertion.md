# Assertion

NBomber provides a way to specify assertions for concreate Step within Scenario.

# [C#](#tab/tabid-1)
```csharp
public class Assertion
{
    public static IAssertion ForStep(string stepName, Func<Statistics, bool> assertion, string label = null);
}
```

# [F#](#tab/tabid-2)
```fsharp
type Assertion =
    static member forStep (stepName: string, assertion: Statistics -> bool, ?label: string)
```
***

## API Usage

- [C# Example](https://github.com/PragmaticFlow/NBomber/blob/dev/examples/CSharp/CSharp.Examples.NUnit/Tests.cs)
- [F# Example](https://github.com/PragmaticFlow/NBomber/blob/dev/examples/FSharp/FSharp.Examples.XUnit/Tests.fs)