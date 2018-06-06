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
[Serializable]
public class RootDialog : IDialog<object>
{

    public async Task StartAsync(IDialogContext context)
    {
        // Wait for the first message from the user
        context.Wait(MessageReceivedAsync);
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> response)
    {
        // Now we received a message
        var message = await response;

        // check if our messages matches some of our actions?
        switch(message.Text.ToLower())
        {
            case "request time-off":
                // Put the dialog ontop of the stack, but the method will still continue!
                context.Call(new RequestTimeOffDialog(), ResumeAfterDialog);
                break;
            case "show time-off":
                context.Call(new ShowTimeOffDialog(), ResumeAfterDialog);
                break;
            case "help":
                context.Call(new HelpDialog(), ResumeAfterDialog);
                break;
            default:
                // Write welcome message and wait for a response
                WriteWelcomeMessage(context);
                context.Wait(MessageReceivedAsync);
                break;
        }
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

    private async Task ResumeAfterDialog(IDialogContext context, IAwaitable<object> result)
    {
        // Display selection menu again
        WriteWelcomeMessage(context);
        context.Wait(this.MessageReceivedAsync);
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

## Alternative Solution using Prompts

Alternatively, we can use `PromptsDialog` in `RootDialog`. This is a more strict selction dialog, that is usually used for asking the user for very specific pieces of information (e.g., a date, quanity, etc.).

```csharp
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
```