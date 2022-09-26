> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

The Adaptive Card action handler feature enables the app to respond to adaptive card actions that triggered by end users to complete a sequential workflow. When user gets an Adaptive Card, it can provide one or more buttons in the card to ask for user's input, do something like calling some APIs, and then send another adaptive card in conversation to response to the card action.

In this tutorial you will learn:
* [How to create a workflow bot](#How-to-create-a-workflow-bot)
* [How to understand the workflow bot project](#Take-a-tour-of-your-app-source-code)
* [How to customize the initialization](#Customize-the-initialization)
* [How to customize the installation](#Customize-installation)
* [How to customize the command logic](#Customize-the-command-logic)
* [How to customize the adaptive card](#Customize-the-Adaptive-Card)
* [How to add more actions and responses](#Add-more-card-actions)
* [How to add notifications to your project](#Add-notifications-to-your-application)
* [How to access Microsoft Graph from your workflow bot](#Access-Microsoft-Graph)
* [How to connect to existing APIs from your workflow bot](#Connect-to-existing-APIs)

## How to create a workflow bot

### In Visual Studio Code

1. From Teams Toolkit sidebar click `Create a new Teams app` or select `Teams: Create a new Teams app` from command palette.

![image](https://user-images.githubusercontent.com/11220663/165435370-99aa79b8-044f-44ea-b2a9-e42a055a3f6c.png)

2. Select `Create a new Teams app`.

![image](https://user-images.githubusercontent.com/11220663/168242250-34ca599f-1c9b-4c0c-80a7-ac07ebe10a1a.png)

3. Select the `Workflow bot` from Scenario-based Teams app section.

![image](https://user-images.githubusercontent.com/11220663/192218730-ea4419c9-6d9d-460e-884e-896e7ad428df.png)

4. Select programming language

![image](https://user-images.githubusercontent.com/11220663/165435816-e6d46074-6e0a-4186-804b-c83bfbe12b6f.png)

5. Enter an application name and then press enter.

![image](https://user-images.githubusercontent.com/11220663/165435852-686deaef-119e-4311-9343-d8ef4b335516.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.
* If you prefer non-interactive mode, enter all required parameters in one command.

`teamsfx new --interactive false --capabilities "workflow-bot" --programming-language "typescript" --folder "./" --app-name myWorkflowBot`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. You can send a `helloWorld` command after running this template and get a response as below:

![image](https://user-images.githubusercontent.com/11220663/192219663-a970b23d-3f11-4b1d-afc8-381d26c67327.png)

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Take a tour of your app source code

This section walks through the generated code. The project folder contains the following:

| Folder | Contents |
| - | - |
| `.fx` | Project level settings, configurations, and environment information |
| `.vscode` | VSCode files for local debug |
| `bot` | The source code for the workflow bot Teams application |
| `templates` | Templates for the Teams application manifest and for provisioning Azure resources |

The core command-response implementation is in `bot` folder.

The following files provide the business logic for the workflow bot. These files can be updated to fit your business logic requirements. The default implementation provides a starting point to help you get started.

| File | Contents |
| - | - |
| `src/index.ts` | Application entry point and `restify` handlers for the workflow bot |
| `src/adaptiveCards/helloworldCommand.json` | A generated Adaptive Card that is sent to Teams |
| `src/commands/helloworldCommandHandler.ts` | Responds to the command message |
| `src/cardActions/doStuffActionHandler.ts` | Responds to the `doStuff` card action |
| `src/cardModels.ts` | The default Adaptive Card data model |

The following files implement the core workflow bot on the Bot Framework. You generally will not need to customize these files.

| File / Folder | Contents |
| - | - |
| `src/internal/initialize.ts` | Application initialization and bot message handling |

The following files are project-related files. You generally will not need to customize these files.

| File / Folder | Contents |
| - | - |
| `.gitignore` | Git ignore file |
| `package.json` | NPM package file |

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Customize the initialization

The default initialization is located in `bot/src/internal/initialize.ts`.

You can update the initialization logic to:

- Set `options.adapter` to use your own `BotFrameworkAdapter`
- Set `options.command.commands` to include more command handlers
- Set `options.cardAction.actions` to include more action handlers
- Set `options.{feature}.enabled` to enable more `ConversationBot` functionality

To learn more, visit [additional initialization customizations](https://aka.ms/teamsfx-command-response#customize-initialization).

## Customize installation
A Teams bot can be installed into a team, or a group chat, or as personal app, depending on difference scopes. You can choose the installation target when adding the App.
- See [Distribute your app](https://docs.microsoft.com/microsoftteams/platform/concepts/deploy-and-publish/apps-publish-overview) for more install options.

  ![Installation Target](notification/addanapp.png)

- See [Remove an app from Teams](https://support.microsoft.com/office/remove-an-app-from-teams-0bc48d54-e572-463c-a7b7-71bfdc0e4a9d) for uninstallation.

## Customize the command logic

The default command logic simply returns a hard-coded Adaptive Card. You can customize this logic with your customize business logic. Often your business logic might require you to call your existing APIs.

Teams Toolkit enables you to [easily connect to an existing API](#connect-to-existing-apis).

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Customize the Adaptive Card

You can edit the file `src/adaptiveCards/helloworldCommand.json` to customize the Adaptive Card to your liking. The file `src/cardModels.ts` defines a data structure that is used to fill data for the Adaptive Card.

The binding between the model and the Adaptive Card is done by name matching (for example,`CardData.title` maps to `${title}` in the Adaptive Card). You can add, edit, or remove properties and their bindings to customize the Adaptive Card to your needs.

You can also add new cards if appropriate for your application. Please follow this [sample](https://aka.ms/teamsfx-adaptive-card-sample) to see how to build different types of adaptive cards with a list or a table of dynamic contents using `ColumnSet` and `FactSet`.

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Add more card actions

You can use the following 4 steps to add more card action:
1. [Step 1: add an action to your Adaptive Card](#step-1-add-an-action-to-your-adaptive-card)
2. [Step 2: add adaptive card for action response](#step-2-add-adaptive-card-for-action-response)
3. [Step 3: add action handler](#step-3-add-action-handler)
4. [Step 4: register the action handler](#step-4-register-the-action-handler)

### Step 1: add an action to your Adaptive Card

Here's a sample action with type `Action.Execute`:
```json
{ 
  "type": "AdaptiveCard", 
  "body": [
    ...
    {
      "type": "ActionSet",
      "actions": [
        {
          "type": "Action.Execute",
          "title": "DoStuff",
          "verb": "doStuff" 
        }
      ]
    }
  ]
  ... 
} 
```

`Action.Execute` invoking the bot can return Adaptive Cards as a response, which will replace the existing card in conversation by default.  

### Step 2: add adaptive card for action response
For each action invoke, you can return a new adaptive card to display the response to end user. You can use [adaptive card designer](https://adaptivecards.io/designer/) to design your card layout according to your business needs.

To get-started, you can just create a sample card (`responseCard.json`) with the following content, and put it in `bot/src/adaptiveCards` folder:

```json
{
  "type": "AdaptiveCard",
  "body": [
    {
      "type": "TextBlock",
      "size": "Medium",
      "weight": "Bolder",
      "text": "This is a sample action response."
    }
  ],
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.4"
}
```

### Step 3: add action handler 

Add handler to implements `TeamsFxAdaptiveCardActionHandler` to process the logic when corresponding action is executed.

Please note:
* The `triggerVerb` is the `verb` property of your action. 
* The `actionData` is the data associated with the action, which may include dynamic user input or some contextual data provided in the `data` property of your action.
* If an Adaptive Card is returned, then the existing card will be replaced with it by default.

```typescript
import { AdaptiveCards } from "@microsoft/adaptivecards-tools";
import { TurnContext, InvokeResponse } from "botbuilder";
import { TeamsFxAdaptiveCardActionHandler, InvokeResponseFactory } from "@microsoft/teamsfx";
import responseCard from "../adaptiveCards/responseCard.json";

export class Handler1 implements TeamsFxAdaptiveCardActionHandler { 
    triggerVerb = "doStuff";

    async handleActionInvoked(context: TurnContext, actionData: any): Promise<InvokeResponse> { 
        const responseCardJson = AdaptiveCards.declare(responseCard).render(actionData);
        return InvokeResponseFactory.adaptiveCard(responseCardJson);
    } 
} 
```

> Note: you can follow [this section](#customize-card-action-handler) to customize the card action handler according to your business need. 

### Step 4: register the action handler

1. Go to `bot/src/internal/initialize.ts`;
2. Update your `conversationBot` initialization to enable cardAction feature and add the handler to `actions` array:

```typescript
export const conversationBot = new ConversationBot({ 
  ... 
  cardAction: { 
    enabled: true, 
    actions: [ 
      new Handler1() 
    ], 
  } 
}); 
```

For more code snippets and details, refer to [this document](https://aka.ms/teamsfx-card-action-response#).

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Add notifications to your application

The notification feature adds the ability for your application to send Adaptive Cards in response to external events. For example, when a message is posted to `Event Hub` your application can respond and send an appropriate Adaptive Card to Teams.

To add the notification feature:

1. Go to `bot\src\internal\initialize.ts`
2. Update your `conversationBot` initialization to enable notification feature:
    ```typescript
    const conversationBot = new ConversationBot({ 
      ... 
      cardAction: { 
        enabled: true, 
        actions: [ 
          new Handler1() 
        ], 
      },
      notification: {
        enabled: true
      } 
    }); 
    ```

3. To quickly add a sample notification triggered by a HTTP request, you can add the following sample code in `bot\src\index.ts`:

    ```typescript
    server.post("/api/notification", async (req, res) => {
      for (const target of await conversationBot.notification.installations()) {
        await target.sendMessage("This is a sample notification message");
      }
    
      res.json({});
    });
    ```

4. Uninstall your previous bot installation from Teams, and press `F5` to start your application.
5. Send a notification to the bot installation targets (channel/group chat/personal chat) by using a your favorite tool to send a HTTP POST request to `https://localhost:3978/api/notification`.

To learn more, refer to [the notification document](https://aka.ms/teamsfx-notification).

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Access Microsoft Graph

If you are responding to a command that needs access to Microsoft Graph, you can leverage single sign on to leverage the logged-in Teams user token to access their Microsoft Graph data. Read more about how Teams Toolkit can help you [add SSO](https://aka.ms/teamsfx-add-sso) to your application.

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>

## Connect to existing APIs

Often you need to connect to existing APIs in order to retrieve data to send to Teams. Teams Toolkit makes it easy for you to configure and manage authentication for existing APIs. 

For more information, [click here](https://aka.ms/teamsfx-connect-api).

<p align="right"><a href="#How-to-create-a-workflow-bot">back to top</a></p>
