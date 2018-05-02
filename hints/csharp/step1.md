# Hints for completing Step 1

## Setting up the Resource Group and Storage Account

Create a new Resource Group for our bot:

```
az group create --location "West Europe" --name chatbotday-csharp
```

Create a new Storage Account for holding the bot's state:
```
az storage account create --resource-group chatbotday-csharp --sku Standard_LRS --name cbdbotstatecs

az storage table create --account-name cbdbotstatecs --name botstateprod

# We'll use a separate table for local testing
az storage table create --account-name cbdbotstatecs --name botstatedev
```

Upon bot creation, Azure automatically creates a Storage Account for you, but given the random naming, it might be less confusing to create your own.

Get the connection string for storage account access:

```
az storage account show-connection-string --resource-group chatbotday-csharp --name cbdbotstatecs
{
  "connectionString": "DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=xxxxx;AccountKey=xxxxxxxxxxxxx"
}
```

## Create Bot Service and setup deployment via git

1. Create Bot Service based on App Service (Web App Bot) with the `Basic C#` template
1. Install [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
1. Install [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
1. Configure local Git repo: Navigate to your bot --> `Build` --> `Configure Continuous Deployment` --> `Setup` --> `Local Git Repository`
1. Retrieve Git Clone URL: `All App Service Settings` --> `Overview` --> `Git Clone URL` (might need to set up password first)

## Clone and install packages (locally)

Next, clone the repo locally:

```
git clone https://<user>@<botname>.scm.azurewebsites.net:443/<botname>.git
cd <botname>
```

You can now use [Visual Studio 2017](https://www.visualstudio.com/downloads/) for easy development, debugging and committing to `git`:

```
start .\Microsoft.Bot.Sample.SimpleEchoBot.sln
```

For setting up publishing back to our Bot Service in Azure, we can right click our bot project `Microsoft.Bot.Sample.SimpleEchoBot` and select `Publish`. Then select `Create new Profile`, select `Existing App Service`, and then pick the Bot Web App we just created. You might need to update your credentials to your deployment, so Visual Studio can push to Git repository.

For a production bot, it obviously makes sense to rename your bot to something more meaningful. As we want to focus on getting our bot and running, we'll skip this step for sake of time today.

## Setup bot state table

Open `Global.asax.cs` and adapt it to use our new storage account and table storage (for bot state):

```csharp
var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["STORAGE_CONNECTION"].ConnectionString, ConfigurationManager.AppSettings["BotStateTable"]);
```

Next, update your `Web.config` with our new connection string:

```xml
  <appSettings>
    <add key="MicrosoftAppId" value="" />
    <add key="MicrosoftAppPassword" value="" />
    <add key="BotStateTable" value="botstatedev" />
  </appSettings>

  <connectionStrings>
    <add name="STORAGE_CONNECTION" connectionString="DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=xxxx;AccountKey=xxxxx"/>
  </connectionStrings>
```

Lastly, go to `Bot Service in Azure Portal` --> `Application Settings` --> `App Settings` and add two new fields

* `BotStateTable` --> `botstateprod` (under App Settings)
* `STORAGE_CONNECTION` --> `DefaultEndpointsProtocol=https;EndpointSuffix=core.windows.net;AccountName=xxx;AccountKey=xxxxx` (under Connection Strings as type `Custom`)

*Note:* There are cleaner ways to keep your credentials out of the code (and code repository in general), e.g., by using [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/) or having separate config files for local testing, which are not committed to the source code repository.

## Test bot locally

Now we should be able to run our bot locally, by pressing the green play button.

* Connect via Bot Framework Emulator to `http://localhost:3984/api/messages` and test if the bot is working
* You won't need to enter the App ID or App Password, since you're just testing it locally
* Check your Table storage via Storage Explorer and see what happened

## Push changes to production 

Commit changes to `git`:

```
git add .
git commit -m "Added table storage"
git push azure
```

This will commit your code to the git repository in Azure and automatically trigger the deployment process. The easiest way to test if your bot works in Azure is by using the `Test in Web Chat` functionality.

Alternatively, you can use the `Publish` functionality in Visual Studio, which will basically perform the same steps for you in the background.

## Add Bot to Microsoft Teams

1. Change name of bot: `Settings` --> `Change Display Name` --> `Your new bot's name` (don't forget to hit `Save`)
1. Add to channel: `Channels` --> `Teams` --> `Enable`
1. Test: Click `Teams` entry --> link opens --> Teams app opens --> Bot should be there and functional (might take a minute or two until the bot starts responding)

Congratulations, you finished the basic bot setup and you're now able to easily push code updates to Azure!
