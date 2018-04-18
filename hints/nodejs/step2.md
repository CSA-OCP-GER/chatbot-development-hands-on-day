# Hints for completing Step 2

## Main dialog with selection buttons

Add main selection dialog with buttons:

```javascript
bot.dialog('/', [
    function (session) {
        session.send("Hello, I'm here to help you with booking time-off requests!");
        builder.Prompts.choice(session, "How can I help you?", ["Request time-off", "Show time-off"], { listStyle: builder.ListStyle.button });
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
    session.send("I'm here to help you:");
    session.endDialog("For example, ask me to 'Request time-off' or 'Show time-off'");
}).triggerAction({
    matches: /^help$/i
});
```
