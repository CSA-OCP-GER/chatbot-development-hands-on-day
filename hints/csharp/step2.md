# Hints for completing Step 2

## Main dialog with selection buttons

*As a side note*: There are many ways of how to implement this, but as we'll enhance the dialog routing logic with LUIS in the next step, let's just get something working quickly.

Open the `Dialogs` folder and add three new files of the type `Bot Dialog`:

* `RootDialog.cs`
* `RequestTimeOffDialog.cs`
* `ShowTimeOffDialog.cs`

## Adding RootDialog

First, we need to make sure our `MessagesController.cs` forwards messages into our `RootDialog.cs`:

```csharp
    [ResponseType(typeof(void))]
    public virtual async Task<HttpResponseMessage> Post([FromBody] Activity activity)
    {
        if (activity != null && activity.GetActivityType() == ActivityTypes.Message)
        {
            await Conversation.SendAsync(activity, () => new RootDialog());
        }
        else
        {
            HandleSystemMessage(activity);
        }
        return new HttpResponseMessage(System.Net.HttpStatusCode.Accepted);
    }
```

## Core dialogs

Now, we can implement placeholders for `RequestTimeOffDialog` and `ShowTimeOffDialog`:

```csharp
namespace Microsoft.Bot.Sample.SimpleEchoBot.Dialogs
{
    [Serializable]
    public class RequestTimeOffDialog : IDialog<object>
    {
        public async Task StartAsync(IDialogContext context)
        {
            await context.PostAsync("Here you will later implement code to request time-off!");
            context.Done<object>(null);
        }
    }
}
```

and

```csharp
namespace Microsoft.Bot.Sample.SimpleEchoBot.Dialogs
{
    [Serializable]
    public class ShowTimeOffDialog : IDialog<object>
    {
        public async Task StartAsync(IDialogContext context)
        {
            await context.PostAsync("Here you will later implement code to show time-off!");
            context.Done<object>(null);
        }
    }
}
```

## Route between dialogs through RootDialog

Lastly, we can implement our `RootDialog`:

```csharp
namespace Microsoft.Bot.Sample.SimpleEchoBot.Dialogs
{
    [Serializable]
    public class RootDialog : IDialog<object>
    {

        private const string RequestTimeOff = "Request time-off";
        private const string ShowTimeOff = "Show time-off";

        public async Task StartAsync(IDialogContext context)
        {
            context.Wait(MessageReceivedAsync);
        }

        private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
        {
            PromptDialog.Choice(
                context,
                this.ResumeAfterSelection,
                new[] { RequestTimeOff, ShowTimeOff },
                "Hello, I'm here to help you with booking time-off requests! How can I help you?", 
                "Sorry, I did not understand you. Please choose one of the options below.");
        }

        private async Task ResumeAfterSelection(IDialogContext context, IAwaitable<string> result)
        {
            try
            {
                var selection = await result;

                switch (selection)
                {
                    case RequestTimeOff:
                        context.Call(new RequestTimeOffDialog(), this.MessageReceivedAsync);
                        break;
                    case ShowTimeOff:
                        context.Call(new ShowTimeOffDialog(), this.MessageReceivedAsync);
                        break;
                }
            } catch (TooManyAttemptsException)
            {
                await this.StartAsync(context);
            }
        }
    }
}
```

## Add help dialog

Last but not least, we can add an helper dialog if desired:

```csharp
namespace Microsoft.Bot.Sample.SimpleEchoBot.Dialogs
{
    [Serializable]
    public class HelpDialog : IDialog<object>
    {
        public async Task StartAsync(IDialogContext context)
        {
            await context.PostAsync("One day, this dialog might help you with something...");
            context.Done<object>(null);
        }
    }
}
```
