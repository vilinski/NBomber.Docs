# Configuration

NBomber supports configuration via JSON files. In order to start using it, you just need to add one line of code to your standard setup.

# [F#](#tab/tabid-1)
```fsharp
NBomberRunner.loadConfig "config.json"
```

# [C#](#tab/tabid-2)
```csharp
NBomberRunner.LoadConfig("config.json")
```
***

## Single node config

```json
{
  "GlobalSettings": {
    "ScenariosSettings": [
      {
        "ScenarioName": "test_youtube",
        "WarmUpDuration": "00:00:05",
        "Duration": "00:00:20",
        "ConcurrentCopies": 100
      },
      {
        "ScenarioName": "test_mongo",
        "WarmUpDuration": "00:00:05",
        "Duration": "00:00:20",
        "ConcurrentCopies": 100
      }
    ],
    "TargetScenarios": [ "test_youtube" ], // will run only one scenario
    "ReportFileName": "custom_report_name_from_json", // optional
    "ReportFormats": [ "Html", "Txt", "Csv", "Md" ] // optional
  }
}
```

## Cluster Coordinator config

```json
{
  "GlobalSettings": {
    "ScenariosSettings": [
      {
        "ScenarioName": "test_youtube",
        "WarmUpDuration": "00:00:05",
        "Duration": "00:00:20",
        "ConcurrentCopies": 100
      },
      {
        "ScenarioName": "test_mongo",
        "WarmUpDuration": "00:00:05",
        "Duration": "00:00:20",
        "ConcurrentCopies": 100
      }
    ],
    "TargetScenarios": [ "test_youtube" ], // will run only one scenario
    "ReportFileName": "custom_report_name_from_json", // optional
    "ReportFormats": [ "Html", "Txt", "Csv", "Md" ] // optional
  },
  
  "ClusterSettings": {
   "Coordinator": {
     "ClusterId": "test_cluster",
     "TargetScenarios": [ "test_youtube" ],
     
     "Agents": [
       {
         "Host": "localhost",
         "Port": 2222,
         "TargetScenarios": [ "test_youtube" ]
       }
     ]
   }
  }
}
```

## Cluster Agent config
```json
{ 
  "ClusterSettings": {      
    "Agent": {
    "ClusterId": "test_cluster",
    "Port": 2222
   }
  }
}
```
