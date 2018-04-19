# Hints for completing Step 2

## Main dialog with selection buttons

*As a side note*: There are many ways of how to implement this, but as we'll enhance the dialog routing logic with LUIS in the next step, let's just get something working quickly.

Add main selection dialog with buttons:

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

Having `replaceDialog` at the end of the main dialog makes it repeat forever. Alternatively, we could ask the user if they want to continue, i.e. closing the conversation if they answer with 'no'.

## Core dialogs

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

## Add help dialog

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("I'm here to help you:<br>For example, ask me to 'Request time-off' or 'Show time-off'");
}).triggerAction({
    matches: /^help$/i
});
```
