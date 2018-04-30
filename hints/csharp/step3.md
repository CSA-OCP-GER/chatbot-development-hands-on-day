# Hints for completing Step 3

## LUIS Model training

1. Create new LUIS app
1. Add `Greeting` intent and write in examples
1. Add `RequestTimeoff` intent and write in examples
1. Add `ShowTimeoff` intent and write in examples
1. Train model
1. Publish model, and note down the LUIS Endpoint URL

## Update code with LUIS model

First, we need to add the LUIS AppId, Key and Hostname to our `Web.config`

```xml
<appSettings>
    ...
    <add key="LuisAppId" value="xxx" />
    <add key="LuisAPIKey" value="xxx" />
    <add key="LuisAPIHostName" value="westus.api.cognitive.microsoft.com" />
</appSettings>
```

Next, update your App Settings in Azure to also have fields and values for `LuisAppId`, `LuisAPIKey`, and `LuisAPIHostName`.

Then, update all relevant dialogs to leverage `triggerAction` for linkage with the LUIS intents:

```csharp

```