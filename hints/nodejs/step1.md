# Hints for completing Step 1

## Setting up the Resource Group and Storage Account

```
az group create --location "West Europe" --name vacation-bot-nodejs

az storage account create --resource-group vacation-bot-nodejs --sku Standard_LRS --name clemensbotdata

az storage table create --account-name clemensbotdata --name botstateprod
az storage table create --account-name clemensbotdata --name botstatedev
```

Get connection string for storage account access:

```
az storage account show-connection-string --resource-group vacation-bot-nodejs --name clemensbotdata
{
  "connectionString": "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=clemensbotdata;AccountKey=xxxxxxxxxxxxx"
}
```

## Create Bot Service and setup deployment via git

1. Create Bot Service based on App Service
1. Install [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
1. Install [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
1. Configure local Git repo: `Build` --> `Configure Continuous Deployment` --> `Setup` --> `Local Git Repo`
1. Retrieve Git Clone URL: `All App Service Settings` --> `Overview` --> `Git Clone URL`

## Clone and install packages (locally)

Clone repo locally and install Node packages:

```
git clone https://<user>@<botname>.scm.azurewebsites.net:443/<botname>.git
cd <botname>
npm install
```

## Setup bot state table

Open `app.js` and adapt it to use our new storage account and table storage (for bot state):

```
var tableName = "botstateprod";
var storageAccountConnection = "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=xxx;AccountKey=xxxx";
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, storageAccountConnection);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);
```
*Important:* See below on how to keep the secrets out of source code!

## Test bot locally

Start bot locally:

```
node app.js
```

* Connect via Bot Framework Emulator and test it
* Check table storage via Storage Explorer and see what happened

## Push changes to production 

Commit changes to `Git`:

```
git add app.js
git commit -m "Added table storage"
git push
```

Bot should now auto deploy. We can now integrate it into Teams:

## Add Bot to Microsoft Teams

1. Change name of bot: `Settings` --> `Change Display Name` --> `Your new bot's name`
1. Add to channel: `Channels` --> `Teams` --> walk through dialogs
1. Test: Click Teams --> link opens --> Teams App opens --> Bot should be there and functional

## Proper setup of secrets

Setup proper env variables via `dotenv` library:

```
npm install dotenv --save
```

Edit `.env`:

```
BOT_STATE_TABLE=botstatedev
STORAGE_ACCOUNT=csvacationbotdata
STORAGE_ACCOUNT_KEY=xxxx
```

Goto `Bot Service in Azure Portal` --> `Application Settings` --> `App Settings` and add two new fields

* `BOT_STATE_TABLE` --> `botstateprod`
* `STORAGE_ACCOUNT_CONNECTION_STRING` --> `DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=xxx;AccountKey=xxxxx`

Update code:

```javascript
require('dotenv').config();
...
var tableName = process.env.BOT_STATE_TABLE;
var storageAccountConnection = process.env.STORAGE_ACCOUNT_CONNECTION_STRING;
```

Add `.env` to `.gitignore` and commit changes to Git:

```
git add .gitignore app.js package.json
git commit -m "Proper storing of secrets in .env file"
git push
```
