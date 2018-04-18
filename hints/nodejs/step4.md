# Hints for completing Step 4

## Welcoming the user

This piece of code triggers when a user joins the conversation and can be used to actively invoke the root dialog:

```javascript
bot.on('conversationUpdate', function (message) {
    if (message.membersAdded && message.membersAdded.length > 0) {
        message.membersAdded.forEach(function (identity) {
            if (identity.id === message.address.bot.id) {
                bot.beginDialog(message.address, '/')
            }
        });
    }
});
```