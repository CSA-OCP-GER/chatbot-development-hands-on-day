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

## Handling time-off requests

Handling time-off requests can either happen through LUIS by recognizing entities or by just guiding the user through a waterfall dialog and asking for the respective fields. In this example, we just use a waterfall dialog and the Bot Framework will already do the job of parsing the date reliably for us.

For sake of simplicity, we're storing the time-off requests in the user's `session.userData`. This is persisted across multiple conversations, but specific per channel. A better solution would be to persist this in a Table, SQL database, in CosmosDB or somewhere else.

```javascript
bot.dialog('requestTimeOff', [
    function (session) {
        if (session.userData.timeoff == null) {
            session.userData.timeoff = [];
        }
        builder.Prompts.time(session, "Please provide a start date (e.g.: June 6th 2018)");
    },
    function (session, results) {
        session.dialogData.startDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.time(session, "Please provide an end date (e.g.: June 8th 2018)");
    },
    function (session, results) {
        session.dialogData.endDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "Where are you heading to?");
    },
    function (session, results) {
        session.dialogData.destination = results.response;
        session.send("Time-off request from %s to %s, %s",
            session.dialogData.startDate, session.dialogData.endDate, session.dialogData.destination);
        builder.Prompts.confirm(session, "Do you want to proceed in this request?", { listStyle: builder.ListStyle.button });
    },
    function (session, results) {
        if (results.response) {
            let timeoffRequest = {
                "start_date": session.dialogData.startDate,
                "end_date": session.dialogData.endDate,
                "destination": session.dialogData.destination
            };
            session.userData.timeoff.push(timeoffRequest)

            session.send('Time-off request was accepted');
        } else {
            session.send('Time-off request was discarded');
        }
        session.endDialog();
    }
]).triggerAction({
    matches: 'RequestTimeOff'
});
```

## Showing time-off requests

Lastly, we can use a [carousel of Hero cards](https://docs.microsoft.com/en-us/azure/bot-service/nodejs/bot-builder-nodejs-send-rich-cards#send-a-carousel-of-hero-cards) to nicely display the existing time-off requests for a user. This would also allow us to easily add a button for deleting existing requests.

```javascript
bot.dialog('requestTimeOff', [
    function (session) {
        if (session.userData.timeoff == null) {
            session.userData.timeoff = [];
        }

        builder.Prompts.time(session, "Please provide a start date (e.g.: June 6th 2018)");
    },
    function (session, results) {
        session.dialogData.startDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.time(session, "Please provide an end date (e.g.: June 8th 2018)");
    },
    function (session, results) {
        session.dialogData.endDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "Where are you heading to?");
    },
    function (session, results) {
        session.dialogData.destination = results.response;
        session.send("Time-off request from %s to %s, %s",
            session.dialogData.startDate, session.dialogData.endDate, session.dialogData.destination);
        builder.Prompts.confirm(session, "Do you want to proceed in this request?", { listStyle: builder.ListStyle.button });
    },
    function (session, results) {
        if (results.response) {
            let timeoffRequest = {
                "start_date": session.dialogData.startDate,
                "end_date": session.dialogData.endDate,
                "destination": session.dialogData.destination
            };
            session.userData.timeoff.push(timeoffRequest)

            session.send('Time-off request was accepted');
        } else {
            session.send('Time-off request was discarded');
        }
        session.endDialog();
    }
]).triggerAction({
    matches: 'RequestTimeOff'
});
```
![Our final bot](../../images/final.jpg "Our final bot")
