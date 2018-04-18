# Hints for completing Step 3

## LUIS Model training

1. Create new LUIS app
1. Add `RequestTimeoff` intent and write in examples
1. Add `ShowTimeoff` intent and write in examples
1. Train model
1. Publish model, and note down LUIS Model URL (including `Subscription-Key`)

## Update code with LUIS model

```javascript
var luisModelURL = process.env.LUIS_MODEL_URL;

var recognizer = new builder.LuisRecognizer(luisModelURL);
bot.recognizer(recognizer);
```

Then, update all relevant dialogs to leverage `triggerAction` for linkage with the LUIS intents:

```javascript
bot.dialog('name', function (session, args) {
  ...
}).triggerAction({
  matches: '<Intent name>'
});
```