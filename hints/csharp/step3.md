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

Next, let's create an `LuisRootDialog.cs` for implementing a new root dialog:

```csharp
[Serializable]
public class LuisRootDialog : LuisDialog<object>
{

    public LuisRootDialog() : base(new LuisService(new LuisModelAttribute(
        ConfigurationManager.AppSettings["LuisAppId"],
        ConfigurationManager.AppSettings["LuisAPIKey"],
        domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
    {
    }

    [LuisIntent("None")]
    public async Task NoneIntent(IDialogContext context, LuisResult result)
    {
        await this.ShowLuisResult(context, result);
    }

    [LuisIntent("Greeting")]
    public async Task GreetingIntent(IDialogContext context, LuisResult result)
    {
        WriteWelcomeMessage(context);
    }

    [LuisIntent("RequestTimeOff")]
    public async Task RequestTimeOffIntent(IDialogContext context, LuisResult result)
    {
        context.Call(new RequestTimeOffDialog(), ResumeAfterDialog);
    }

    [LuisIntent("ShowTimeOff")]
    public async Task ShowTimeOffIntent(IDialogContext context, LuisResult result)
    {
        context.Call(new ShowTimeOffDialog(), ResumeAfterDialog);
    }

    private async Task ShowLuisResult(IDialogContext context, LuisResult result)
    {
        await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
        context.Wait(MessageReceived);
    }

    private async Task ResumeAfterDialog(IDialogContext context, IAwaitable<object> result)
    {
        WriteWelcomeMessage(context);
        context.Wait(MessageReceived);
    }

    private async void WriteWelcomeMessage(IDialogContext context)
    {
        var activity = context.Activity as Activity;

        var reply = activity.CreateReply("Hello, I'm here to help you with booking time-off requests! How can I help you?");
        reply.Type = ActivityTypes.Message;
        reply.TextFormat = TextFormatTypes.Plain;

        reply.SuggestedActions = new SuggestedActions()
        {
            Actions = new List<CardAction>()
            {
                new CardAction(){ Title = "Request time-off", Type=ActionTypes.ImBack, Value="Request time-off" },
                new CardAction(){ Title = "Show time-off", Type=ActionTypes.ImBack, Value="Show time-off" }
            }
        };
        await context.PostAsync(reply);
    }
}
```

Lastly, update `MessagesController.cs` to route to our new `LuisRootDialog`:

```csharp
[ResponseType(typeof(void))]
public virtual async Task<HttpResponseMessage> Post([FromBody] Activity activity)
{
    // check if activity is of type message
    if (activity != null && activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new LuisRootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    return new HttpResponseMessage(System.Net.HttpStatusCode.Accepted);
}
```
