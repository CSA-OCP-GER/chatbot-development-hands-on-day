# Hints for completing Step 3

**Common pitfalls in this step**
> * Ensure `matches: 'IntentName'` matches exactly the LUIS intent name (case sensitive!)
> * Make sure your LUIS `None` intent has some utterances (Reason [see here](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-intent#none-intent-is-fallback-for-app))

## LUIS Model training

1. Create new LUIS app
1. Add `Greeting` intent and write in examples
1. Add `RequestTimeOff` intent and write in examples
1. Add `ShowTimeOff` intent and write in examples
1. Train model
1. Publish model, and note down the complete LUIS Endpoint URL (including `Subscription-Key`)

## Update code with LUIS model

```javascript
const LUIS_MODEL_URL = process.env.LUIS_MODEL_URL;

var recognizer = new builder.LuisRecognizer(LUIS_MODEL_URL);
bot.recognizer(recognizer);
```

Then, update all relevant dialogs to leverage `triggerAction` for linkage with the LUIS intents:

```javascript
bot.dialog('requestTimeOff', [
    ...
]).triggerAction({
    matches: 'RequestTimeOff'
});

bot.dialog('showTimeOff', [
    ...
]).triggerAction({
    matches: 'ShowTimeOff'
});
```

Lastly, update your App Settings in Azure to also have a `LUIS_MODEL_URL` field pointing to the LUIS URL.