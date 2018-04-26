# Hints for completing Step 2

## Main dialog with selection buttons

*As a side note*: There are many ways of how to implement this, but as we'll enhance the dialog routing logic with LUIS in the next step, let's just get something working quickly.

Add main selection dialog with buttons:

```javascript
bot.dialog('/', [
    function (session) {
        session.send("Hello, I'm here to help you with booking time-off requests!");
        var msg = new builder.Message(session)
            .text("How can I help you?")
            .suggestedActions(
                builder.SuggestedActions.create(
                    session, [
                        builder.CardAction.imBack(session, "Request Time-Off", "Request Time-Off"),
                        builder.CardAction.imBack(session, "Show Time-Off", "Show Time-Off"),
                    ]
                ));
        session.send(msg);
    },
    function (session) {
        session.replaceDialog('/');
    }
]);
```

Having `replaceDialog` at the end of the main dialog makes it repeat forever. Alternatively, we could ask the user if they want to continue, i.e. closing the conversation if they answer with 'no'.

## Core dialogs

```javascript
bot.dialog('requestTimeOff', [
    function (session) {
        session.endDialog('Here you will later implement code to request time-off');
    }
]).triggerAction({
  matches: /^request time-off$/i
});

bot.dialog('showTimeOff', [
    function (session) {
        session.endDialog('Here you will later implement code to show time-off');
    }
]).triggerAction({
  matches: /^show time-off$/i
});
```

## Add help dialog

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("I'm here to help you:<br>For example, ask me to 'Request time-off' or 'Show time-off'");
}).triggerAction({
    matches: /^help$/i
});
```

## Alternative Solution

As an alternative, we can also use the `Prompts` helpers:

```javascript
var menuChoices = {
    'Request Time-Off': 'requestTimeOff',
    'Show Time-Off': 'showTimeOff'
};

bot.dialog('/', [
    function(session) {
        session.send("Hello, I'm here to help you with booking time-off requests!");
        builder.Prompts.choice(session, "How can I help you?", Object.keys(menuChoices), { listStyle: builder.ListStyle.button });
    },
    function (session, results) {
        var action = menuChoices[results.response.entity];
        return session.beginDialog(action);
    },
    function (session) {
        session.replaceDialog('/');
    }
]);
```

In this case, we're actively routing answers to the respective dialogs, thus not needing the `triggerAction`:

```javascript
bot.dialog('requestTimeOff', [
    function (session) {
        session.endDialog('Here you will later implement code to request time-off');
    }
]);

bot.dialog('showTimeOff', [
    function (session) {
        session.endDialog('Here you will later implement code to show time-off');
    }
]);
```