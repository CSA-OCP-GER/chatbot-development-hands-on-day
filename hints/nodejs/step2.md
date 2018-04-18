# Hints for completing Step 2

## Main dialog with selection buttons

Add main selection dialog with buttons:

```js
var botChoices = {
    'Request Time-Off': 'requestTimeOff',
    'Show Time-Off': 'showTimeOff'
};

bot.dialog('/', [
    function(session) {
        session.send("Hello, I'm here to help you with booking time-off requests!");
        builder.Prompts.choice(session, "How can I help you?", Object.keys(botChoices), { listStyle: builder.ListStyle.button });
    },
    function (session, results) {
        var action = botChoices[results.response.entity];
        return session.beginDialog(action);
    },
    function (session) {
        session.replaceDialog('/');
    }
]);
```

## Core dialogs

```js
bot.dialog('requestTimeOff', [
    function(session) {
        session.endDialog('Here you will later implement code to request time off');
    }
]);

bot.dialog('showTimeOff', [
    function(session) {
        session.endDialog('Here you will later implement code to show time off');
    }
]);
```

## Add help dialog

```js
bot.dialog('help', function (session, args, next) {
    session.send("I'm here to help you:");
    session.endDialog("For example, ask me to 'Request time-off' or 'Show time-off'");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        session.beginDialog(args.action, args);
    }
});
```
