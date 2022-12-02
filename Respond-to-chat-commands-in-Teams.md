> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

Microsoft Teams allows you to automate simple and repetitive tasks right inside a conversation. You can build a Teams bot that can respond to simple commands sent in chats with adaptive cards.

In this tutorial, you will learn:

Get started with Teams Toolkit and TeamsFx SDK:
* [How to create a command bot with Teams Toolkit](#How-to-create-a-command-response-bot)
* [How to understand the command bot project](#Take-a-tour-of-your-app-source-code)
* [How to understand the command handler](#How-command-and-response-works)

Customize the scaffolded app template:
* [How to customize the initialization](#Customize-initialization)
* [How to customize the installation](#Customize-installation)
* [How to customize your app to add more command and response](#How-to-add-more-command-and-response)
* [How to customize the trigger pattern](#Customize-the-trigger-pattern)
* [How to build dynamic content in response with adaptive cards](#How-to-build-command-response-using-adaptive-card-with-dynamic-content)

Connect your app with Graph or other APIs:
* [How to access Microsoft Graph from your workflow bot](#Access-Microsoft-Graph)
* [How to connect to existing APIs from your command bot](#Connect-to-existing-APIs)

Extend command bot to other bot scenarios:
* [How to extend command bot with notification](#how-to-extend-my-command-and-response-bot-to-support-notification)
* [How to extend command bot with workflow](#how-to-extend-my-command-bot-to-support-adaptive-card-actions)

## How to create a command-response bot

### In Visual Studio Code

1. From Teams Toolkit sidebar click `Create a new Teams app` or select `Teams: Create a new Teams app` from command palette.

![image](https://user-images.githubusercontent.com/11220663/165435370-99aa79b8-044f-44ea-b2a9-e42a055a3f6c.png)

2. Select `Create a new Teams app`.

![image](https://user-images.githubusercontent.com/11220663/168242250-34ca599f-1c9b-4c0c-80a7-ac07ebe10a1a.png)

3. Select the `Command bot` from Scenario-based Teams app section.

![image](https://user-images.githubusercontent.com/11220663/168245299-91749fa4-00b6-4466-89bf-6f35b105828e.png)

4. Select programming language

![image](https://user-images.githubusercontent.com/11220663/165435816-e6d46074-6e0a-4186-804b-c83bfbe12b6f.png)

5. Enter an application name and then press enter.

![image](https://user-images.githubusercontent.com/11220663/165435852-686deaef-119e-4311-9343-d8ef4b335516.png)

### In Visual Studio
1. Make sure you have installed ASP.NET workloads and "Microsoft Teams development tools".
![image](https://user-images.githubusercontent.com/25972542/180788786-95a98752-1207-41e8-9095-613a6b21b78d.png)

2. Create a new project and select "Microsoft Teams App".
![image](https://user-images.githubusercontent.com/25972542/180789311-d0d74a1a-b27e-4d79-b26c-1a839ec48371.png)

3. In next window enter your project name.

4. In next window select Command Bot.
![image](https://user-images.githubusercontent.com/25972542/180798463-eb91c825-aada-48bb-8f7c-83b34d46bc9e.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.

* If you prefer non-interactive mode, enter all required parameters in one command.

`teamsfx new --interactive false --capabilities "command-bot" --programming-language "typescript" --folder "./" --app-name myAppName`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. You can send a `helloWorld` command after running this template and get a response as below:

![command-response](https://user-images.githubusercontent.com/11220663/165891754-16916b68-c1b5-499d-b6a8-bdfb195f1fd0.png)

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Take a tour of your app source code

### For JS/TS project (In Visual Studio Code)
After scaffolding or adding a command-response bot, you will find your bot's source code under `bot/` folder:
| File name | Contents |
|- | -|
|`bot/src/internal/initialize.ts`| Create the global bot adapter and initialize with Teams Frameworks|
|`bot/src/index.ts`| Server code to host the bot app and listen on `/api/messages` to process Teams activity with Bot Framework |
|`bot/src/helloworldCommandHandler.ts`| A hello world command handler to process a `helloworld` command and return an adaptive card as response |
|`bot/src/adaptiveCards/*.json`| Adaptive card JSON file used as your command response |

### For CSharp project (In Visual Studio)
| File name | Contents |
|- | -|
| `Properties` | LaunchSetting file for local debug |
| `.fx` | Project level settings and configurations |
| `Commands` | Define the commands and how your Bot will react to these commands |
| `Controllers` | BotController and NotificationControllers to handle the conversation with user |
| `Models` | Adaptive card data models |
| `Resources` | Adaptive card templates |
| `Templates` | Templates for Teams app manifest and corresponding Azure resources |
| `appsettings` | The Bot settings |
| `GettingStarted.txt` | Instructions on minimal steps to wonderful|
| `Program.cs` | Create the Teams Bot instance |
| `TeamsBot.cs` | An empty Bot handler |

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## How command-and-response works
The TeamsFx Command-Response Bots are created using the [Bot Framework SDK](https://docs.microsoft.com/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0). The Bot Framework SDK provides [built-in message handler](https://docs.microsoft.com/microsoftteams/platform/bots/bot-basics?tabs=javascript#teams-activity-handlers) to handle the incoming message activity, which requires learning curve to understand the concept of Bot Framework (e.g. the [event-driven conversation model](https://docs.microsoft.com/azure/bot-service/bot-activity-handler-concept?view=azure-bot-service-4.0&tabs=javascript)). To simplify the development, the TeamsFx SDK provides command-response abstraction layer to let developers only focus on the development of business logic to handle the command request without learning the Bot Framework SDK.

Behind the scenes, the TeamsFx SDK leverages [Bot Framework Middleware](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0) to handle the integration with the underlying activity handlers. This middleware handles the incoming message activity and invokes the corresponding `handlerCommandReceived` function if the received message text matches the command pattern provided in a `TeamsFxBotCommandHandler` instance. After processing, the middleware will call `context.sendActivity` to send the command response returned from the `handlerCommandReceived` function to the user.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Customize initialization
You can initialize with your own adapter or customize after initialization.

``` typescript
// Create your own adapter
const adapter = new BotFrameworkAdapter(...);

// Customize your adapter, e.g., error handling
adapter.onTurnError = ...

const bot = new ConversationBot({
    // use your own adapter
    adapter: adapter;
    ...
});

// Or, customize later
bot.adapter.onTurnError = ...
```

## Customize installation
A Teams bot can be installed into a team, or a group chat, or as personal app, depending on difference scopes. You can choose the installation target when adding the App.
- See [Distribute your app](https://docs.microsoft.com/microsoftteams/platform/concepts/deploy-and-publish/apps-publish-overview) for more install options.

  ![Installation Target](notification/addanapp.png)

- See [Remove an app from Teams](https://support.microsoft.com/office/remove-an-app-from-teams-0bc48d54-e572-463c-a7b7-71bfdc0e4a9d) for uninstallation.

## How to add more command and response

You can use the following 4 steps to add more command and response:
1. [Step 1: Add a command definition in manifest](#Step-1-Add-a-command-definition-in-manifest)
1. [Step 2: Respond with an Adaptive Card](#Step-2-Respond-with-an-Adaptive-Card)
1. [Step 3: Handle the command](#Step-3-Handle-the-command)
1. [Step 4: Register the new command](#Step-4-Register-the-new-command)

### Step 1: Add a command definition in manifest

You can edit the manifest template file `templates\appPackage\manifest.template.json` to include:
    * The command `title` that user type in the message compose area to trigger the command.
    * The `command` description for this command.

![manifest-add-command](https://user-images.githubusercontent.com/10163840/160374446-7fd164d6-63c9-47b2-9bf1-0d6a88731e8d.png)

### Step 2: Respond with an Adaptive Card

You can build your response data in text format or follow the steps bellow to use adaptive card to render rich content in Teams:

* Prepare your adaptive card content in a JSON file（e.g. myCard.json) under the `bot/adaptiveCards` folder, here is a sample adaptive card JSON payload:

    ```json
            {
                "type": "AdaptiveCard",
                "body": [
                {
                    "type": "TextBlock",
                    "size": "Medium",
                    "weight": "Bolder",
                    "text": "Your Hello World Bot is Running"
                },
                {
                    "type": "TextBlock",
                    "text": "Congratulations! Your hello world bot is running. Click the documentation below to learn more about Bots and the Teams Toolkit.",
                    "wrap": true
                }
                ],
                "actions": [
                {
                    "type": "Action.OpenUrl",
                    "title": "Bot Framework Docs",
                    "url": "https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0"
                },
                {
                    "type": "Action.OpenUrl",
                    "title": "Teams Toolkit Docs",
                    "url": "https://aka.ms/teamsfx-docs"
                }
                ],
                "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                "version": "1.4"
            }
    ```
          
* Import your card content into your code file where your command handler exists: `import myCard from "./adaptiveCards/myCard.json"`

* In your `handleCommandReceived` API, use `MessageBuilder.attachAdaptiveCardWithoutData` or `MessageBuilder.attachAdaptiveCard` to build a bot message activity with the adaptive card and return the message. `return MessageBuilder.attachAdaptiveCardWithoutData(myCard);`

> Note: If you'd like to send adaptive card with dynamic data, please refer to [this section](#how-to-build-command-response-using-adaptive-card-with-dynamic-content).

### Step 3: Handle the command

1. Add a .ts/.js file (e.g. `xxxCommandHandler.ts`) under `bot/src` to handle your bot command, and include the following boilerplate code to get-started:

    ```typescript
    import { Activity, TurnContext } from "botbuilder";
    import { CommandMessage, TeamsFxBotCommandHandler, TriggerPatterns } from "@microsoft/teamsfx";
    import { MessageBuilder } from "@microsoft/teamsfx";

    export class xxxCommandHandler implements TeamsFxBotCommandHandler {
        triggerPatterns: TriggerPatterns = "<string or RegExp pattern to trigger the command>";

        async handleCommandReceived(
            context: TurnContext,
            message: CommandMessage
        ): Promise<string | Partial<Activity>> {
            // verify the command arguments which are received from the client if needed.
            console.log(`Bot received message: ${message.text}`);

            // do something to process your command and return message activity as the response.
            // You can leverage `MessageBuilder` utilities from the `@microsoft/teamsfx` SDK 
            // to facilitate building message with cards supported in Teams.
        }    
    }
    ```

1. Provide the `triggerPatterns` that can trigger this command handler. Usually it's the command name defined in your manifest, or you can use RegExp to handle a complex command (e.g. with some options in the command message).

1. Implement `handleCommandReceived` to handle the command and return a response that will be used to notify the end users. 
* You can retrieve useful information for the conversation from the `context` parameter if needed.
* Parse command input if needed: 
  * `message.text`: the use input message
  * `message.matches`: the capture groups if you uses the RegExp for `triggerPatterns` to trigger the command.

### Step 4: Register the new command

Open `bot\src\internal\initialize.ts` and update the call to `ConversationBot` constructor to include your new added command handlers.

```typescript
export const commandBot = new ConversationBot({
    // The bot id and password to create BotFrameworkAdapter.
    // See https://aka.ms/about-bot-adapter to learn more about adapters.
    adapterConfig: {
        appId: process.env.BOT_ID,
        appPassword: process.env.BOT_PASSWORD,
    },
    command: {
        enabled: true,
        commands: [ new HelloWorldCommandHandler(), new xxxCommandHandler() ],
    },
});
```


Or, use `registerCommand`/ `registerCommands` API from your `ConversationBot` instance to incrementally add your command(s) to a command bot.

```
// register a single command
commandBot.command.registerCommand(new xxxCommandHandler());

// register a set of commands
commandBot.command.registerCommands([
    new xxxCommandHandler(),
    new yyyCommandHandler()
]);
```

Now, you are all done with the code development of adding a new command and response into your bot app. You can just press `F5` to local debug with the command-response bot, or use provision and deploy command to deploy the change to Azure.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Customize the trigger pattern

The default pattern to trigger a command is through a defined keyword. But often times you would want to collect and process additional information retrieved after the trigger keyword. In addition to keyword match, you could also define your trigger pattern with [regular expressions](https://regex101.com/) and match against `message.text` with more controls.

When using regular expressions, any capture group can be found in `message.matches`. Below is an example that uses regular expression to capture strings after `reboot`, for example if user inputs `reboot myMachine`, `message.matches[1]` will capture `myMachine`:

```javascript
class HelloWorldCommandHandler {
  triggerPatterns = /^reboot (.*?)$/i; //"reboot myDevMachine";
  async handleCommandReceived(context, message) {
    console.log(`Bot received message: ${message.text}`);
    const machineName = message.matches[1];
    console.log(machineName);
    // Render your adaptive card for reply message
    const cardData = {
      title: "Your Hello World Bot is Running",
      body: "Congratulations! Your hello world bot is running. Click the button below to trigger an action.",
    };
    const cardJson = AdaptiveCards.declare(helloWorldCard).render(cardData);
    return MessageFactory.attachment(CardFactory.adaptiveCard(cardJson));
  }
}

module.exports = {
  HelloWorldCommandHandler,
}
```

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

### How to build command response using adaptive card with dynamic content?
Adaptive card provides [Template Language](https://docs.microsoft.com/adaptive-cards/templating/) to allow users to render dynamic content with the same layout (the template). For example, use the adaptive card to render a list of items (todo items, assigned bugs, etc) that could varies according to different user. 

1. Add your adaptive card template JSON file under `bot/adaptiveCards` folder
1. Import the card template to you code file where your command handler exists (e.g. `myCommandHandler.ts`)
1. Model your card data
1. Use `MessageBuilder.attachAdaptiveCard` to render the template with dynamic card data

You can also add new cards if appropriate for your application. Please follow this [sample](https://aka.ms/teamsfx-adaptive-card-sample) to see how to build different types of adaptive cards with a list or a table of dynamic contents using `ColumnSet` and `FactSet`.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Access Microsoft Graph

If you are responding to a command that needs access to Microsoft Graph, you can leverage single sign on to leverage the logged-in Teams user token to access their Microsoft Graph data. Read more about how Teams Toolkit can help you [add SSO](https://aka.ms/teamsfx-add-sso) to your application.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Connect to existing API

If you want to invoke external APIs in your code but do not have the appropriate SDK, the "Teams: Connect to an API" command in Teams Toolkit VS Code extension or "teamsfx add api-connection" command in TeamsFx CLI would be helpful to bootstrap code to call target APIs. For more information, you can visit [Connect existing API document](https://aka.ms/teamsfx-connect-api).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Frequently Asked Questions

### How to extend my command and response bot to support notification?

1. Go to `bot\src\internal\initialize.ts(js)`, update your `conversationBot` initialization to enable notification feature:

    ![enable-notification](https://user-images.githubusercontent.com/10163840/165462039-12bd4f61-3fc2-4fc8-8910-6a4b1e138626.png)

2. Follow [this instruction](https://aka.ms/teamsfx-notification#notify) to send notification to the bot installation target (channel/group chat/personal chat). To quickly add a sample notification triggered by a HTTP request, you can add the following sample code in `bot\src\index.ts(js)`:

    ```typescript
    server.post("/api/notification", async (req, res) => {
      for (const target of await commandBot.notification.installations()) {
        await target.sendMessage("This is a sample notification message");
      }
    
      res.json({});
    });

3. Uninstall your previous bot installation from Teams, and re-run local debug to test your bot notification. Then you can send a notification to the bot installation targets (channel/group chat/personal chat) by using a HTTP POST request with target URL `https://localhost:3978/api/notification`.

To explore more details of the notification feature (e.g. send notification with adaptive card, add more triggers), you can further refer to [the notification document](https://aka.ms/teamsfx-notification).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

### How to extend my command bot to support adaptive card actions

The Adaptive Card action handler feature enables the app to respond to adaptive card actions that triggered by end users to complete a sequential workflow. When user gets an Adaptive Card, it can provide one or more buttons in the card to ask for user's input, do something like calling some APIs, and then send another adaptive card in conversation to response to the card action.

To add adaptive card actions to command bot, you can follow the steps [here](https://aka.ms/teamsfx-card-action-response#add-more-card-actions).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>