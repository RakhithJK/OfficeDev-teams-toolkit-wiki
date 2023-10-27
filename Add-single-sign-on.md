# Add single sign on experience

> This feature is currently under active development. Report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

Microsoft Teams provides a mechanism by which an application can obtain the signed-in Teams user token to access Microsoft Graph (and other APIs). Teams Toolkit facilitates this interaction by abstracting some of the Microsoft Entra flows and integrations behind some simple APIs. This enables you to add single sign-on experiences (SSO) easily to your Teams application.

For applications that interact with the user in a chat, a Team, or a channel, SSO manifests as an Adaptive Card which the user can interact with to invoke the Microsoft Entra consent flow.

In this tutorial, you will:

* [Enable SSO for your project](#How-to-enable-sso)
* [Understand what will be changed for your project](#Understanding-the-changes-Teams-Toolkit-makes-to-your-project)
* [Update your code and debug your application](#What-you-need-to-do-after-triggering-Add-SSO-command)
  * [Update code for a tab project](#Update-your-source-code-for-Tab-project)
  * [Update code for a bot project](#Update-your-source-code-for-Bot-project)
* [SSO authentication concepts](#SSO-authentication-concepts)
  * [How SSO works in Teams](#How-SSO-works-in-Teams)
  * [How SSO is simplified with Teams Framework](#How-SSO-are-simplified-with-TeamsFx)

## How to enable SSO

Teams Toolkit can add SSO to the following Teams capabilities: 

* Tab
* Bot
* Notification bot (restify server)
* Command bot

To enable SSO support:

### In Visual Studio Code
1. Open the Teams Toolkit panel ![image](https://user-images.githubusercontent.com/11220663/165348935-9838d8d4-bd7f-409f-ad25-7479fc64a8bf.png)

2. Select `Add features` or open Visual Studio Code Command Palette and select `Teams: Add features`

![image](https://user-images.githubusercontent.com/11220663/165349151-bf009c88-907a-4fd1-9d2d-cdb26e2a93cc.png)

3. Select `Single Sign-On`

![image](https://user-images.githubusercontent.com/11220663/165349533-1227b49d-3df7-4e1b-8d09-202c93b29e9f.png)

### In TeamsFx CLI: 

Run the `teamsfx add sso` command in your project root directory.

> Note: This feature will enable SSO for all existing applicable capabilities. If you add an capability later to the project, you can follow the same steps to enable SSO for that capability.

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

## Understanding the changes Teams Toolkit makes to your project

Teams Toolkit will make the following changes to your project:

|Type| File | Purpose |
|-| - | - |
|Create| `aad.template.json` under `templates/appPackage` | This is the Microsoft Entra application manifest. This template defines your application registration. Your application is registered during provisioning.|
|Modify | `manifest.template.json` under `templates/appPackage` | The Toolkit adds a `webApplicationInfo` definition to the Teams app manifest template. This field is required by Teams when enabling SSO. This change will take effect when provisioning the application.|
|Create| `auth/tab` | Reference code and auth redirect pages for a tab project. |
|Create| `auth/bot` | Reference code and auth redirect pages for a bot project. |

> Note: Adding SSO only makes changes to your project. These changes will be applied to Azure when you provision your application. You also need to update your code to light up the SSO feature. 

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

## Updating your application to use SSO

After adding the SSO feature, follow these steps to enable SSO in your application.

*Note: These changes are based on the templates we scaffold.*

### Tab applications

1. Move `auth-start.html` and `auth-end.htm`** in `auth/public` folder to `tabs/public/`. Teams Toolkit registers these two endpoints in Microsoft Entra for Microsoft Entra's redirect flow.

1. Move `sso` folder under `auth/tab` to `tabs/src/sso/`.

    * `InitTeamsFx`: This file implements a function that initialize TeamsFx SDK and will open `GetUserProfile` component after SDK is initialized.

    * `GetUserProfile`: This file implements a function that calls Microsoft Graph API to get user info.

2. Execute `npm install @microsoft/teamsfx-react` under `tabs/`
3. Add the following lines to `tabs/src/components/sample/Welcome.tsx` to import `InitTeamsFx`:
    ```
    import { InitTeamsFx } from "../../sso/InitTeamsFx";
    ```
4. Replace the following line: `<AddSSO />` with `<InitTeamsFx />` to replace the `AddSso` component with `InitTeamsFx` component.

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

### Bot applications

1. Move `auth/bot/public` folder to `bot/src`. 
This folder contains HTML pages used for Microsoft Entra's redirect flow. *Note*: you need to modify `bot/src/index` file to add routing to these pages.

2. Move `auth/bot/sso` folder to `bot/src`.
These folder contains three files as reference for sso implementation:
    * `showUserInfo`: This implements a function to get user info with SSO token. You can follow this method and create your own method that requires SSO token.
    * `ssoDialog`: This creates a [ComponentDialog](https://docs.microsoft.com/en-us/javascript/api/botbuilder-dialogs/componentdialog?view=botbuilder-ts-latest) that used for SSO.
    * `teamsSsoBot`: This create a [TeamsActivityHandler](https://docs.microsoft.com/en-us/javascript/api/botbuilder/teamsactivityhandler?view=botbuilder-ts-latest) with `ssoDialog` and add `showUserInfo` as a command that can be triggered. 

3. Execute the following commands under `bot/`: `npm install isomorphic-fetch`
4. Execute the following commands under `bot/`: `npm install copyfiles` and replace following line in package.json:
    ```
    "build": "tsc --build",
    ```
    with:
    ```
    "build": "tsc --build && copyfiles public/*.html lib/",
    ```
    This will copy the HTML files used for the Microsoft Entra redirect flow during build.

5. Create a new `teamsSsoBot` instance in `bot/src/index` file. Replace the following code:
    ```
    // Process Teams activity with Bot Framework.
    server.post("/api/messages", async (req, res) => {
        await commandBot.requestHandler(req, res);
    });
    ```

    with:

    ```
    const handler = new TeamsSsoBot();
    // Process Teams activity with Bot Framework.
    server.post("/api/messages", async (req, res) => {
        await commandBot.requestHandler(req, res, async (context)=> {
            await handler.run(context);
        });
    });
    ```

1. Add the HTML routes in the `bot/src/index` file:

    ```
    server.get(
        "/auth-*.html",
        restify.plugins.serveStatic({
            directory: path.join(__dirname, "public"),
        })
    );
    ```

1. Add the following lines to `bot/src/index` to import `teamsSsoBot` and `path`:

    ```
    // For ts:
    import { TeamsSsoBot } from "./sso/teamsSsoBot";
    const path = require("path");

    // For js:
    const { TeamsSsoBot } = require("./sso/teamsSsoBot");
    const path = require("path");
    ```

1. Register your command in the Teams app manifest. Open `templates/appPackage/manifest.template.json`, and add following lines under `command` in `commandLists` of your bot:

    ```
    {
        "title": "show",
        "description": "Show user profile using Single Sign On feature"
    }
    ```
<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

#### (Optional) Add a new command to the bot

After adding SSO in your application, you can also add a new command for your bot to handle.

1. Create a new file (e.g. `todo.ts` or `todo.js`) under `bot/src/` and add your own business logic to call Graph API:

    ```TypeScript
    // for TypeScript:
    export async function showUserImage(
        context: TurnContext,
        ssoToken: string,
        param: any[]
    ): Promise<DialogTurnResult> {
        await context.sendActivity("Retrieving user photo from Microsoft Graph ...");

        // Init TeamsFx instance with SSO token
        const teamsfx = new TeamsFx().setSsoToken(ssoToken);

        // Update scope here. For example: Mail.Read, etc.
        const graphClient = createMicrosoftGraphClient(teamsfx, param[0]);
        
        // You can add following code to get your photo:
        // let photoUrl = "";
        // try {
        //   const photo = await graphClient.api("/me/photo/$value").get();
        //   photoUrl = URL.createObjectURL(photo);
        // } catch {
        //   // Could not fetch photo from user's profile, return empty string as placeholder.
        // }
        // if (photoUrl) {
        //   await context.sendActivity(
        //     `You can find your photo here: ${photoUrl}`
        //   );
        // } else {
        //   await context.sendActivity("Could not retrieve your photo from Microsoft Graph. Please make sure you have uploaded your photo.");
        // }

        return;
    }
    ```

    ```javascript
    // for JavaScript:
    export async function showUserImage(context, ssoToken, param) {
        await context.sendActivity("Retrieving user photo from Microsoft Graph ...");
    
        // Init TeamsFx instance with SSO token
        const teamsfx = new TeamsFx().setSsoToken(ssoToken);
    
        // Update scope here. For example: Mail.Read, etc.
        const graphClient = createMicrosoftGraphClient(teamsfx, param[0]);
        
        // You can add following code to get your photo:
        // let photoUrl = "";
        // try {
        //   const photo = await graphClient.api("/me/photo/$value").get();
        //   photoUrl = URL.createObjectURL(photo);
        // } catch {
        //   // Could not fetch photo from user's profile, return empty string as placeholder.
        // }
        // if (photoUrl) {
        //   await context.sendActivity(
        //     `You can find your photo here: ${photoUrl}`
        //   );
        // } else {
        //   await context.sendActivity("Could not retrieve your photo from Microsoft Graph. Please make sure you have uploaded your photo.");
        // }
    
        return;
    }
    ```

1. Register a new command using `addCommand` in `teamsSsoBot`:

    Find the following line:

    ```
    this.dialog.addCommand("ShowUserProfile", "show", showUserInfo);
    ```

    and add the following lines to register a new command `photo` and connect it with the method `showUserImage`:

    ```
    // As shown here, you can add your own parameter into the `showUserImage` method
    // You can also use regular expression for the command here
    const scope = ["User.Read"];
    this.dialog.addCommand("ShowUserPhoto", new RegExp("photo\s*.*"), showUserImage, scope);
    ```

1. Register your command in the Teams application manifest. Open 'templates/appPackage/manifest.template.json' and add following lines under `command` in `commandLists`:

    ```
    {
        "title": "photo",
        "description": "Show user photo using Single Sign On feature"
    }
    ```

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

## Debug your application

You can debug your application by pressing F5.

Teams Toolkit will use the Microsoft Entra manifest file to register the Microsoft Entra application for SSO.

To learn more about Teams Toolkit local debugging, refer to this [document](https://docs.microsoft.com/microsoftteams/platform/toolkit/debug-local).

## Customize the Microsoft Entra application registration

The Microsoft Entra [manifest](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest) allows you to customize various aspects of your application registration. You can update the manifest as needed.

Follow this [document](https://aka.ms/teamsfx-aad-manifest#Microsoft-Entra-manifest-template) if you need to include additional API permissions to access your desired APIs.

Follow this [document](https://aka.ms/teamsfx-aad-manifest#How-to-view-the-Microsoft-Entra-app-on-the-Azure-portal) to view your Microsoft Entra application in Azure Portal.


## SSO authentication concepts
### How SSO works in Teams

Single sign-on (SSO) authentication in Microsoft Entra silently refreshes the authentication token to minimize the number of times users need to enter their sign in credentials. If users agree to use your app, they don't have to provide consent again on another device as they're signed in automatically. 

Teams Tabs and bots have similar flows for SSO. To learn more, refer to:
* [Use SSO authentication in Tabs](https://docs.microsoft.com/en-us/microsoftteams/platform/tabs/how-to/authentication/auth-aad-sso?tabs=dotnet) 
* [Use SSO authentication in Bots](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/how-to/authentication/auth-aad-sso-bots)

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>

### How Teams Toolkit simplifies bringing SSO to your application

Teams Toolkit helps to reduce the developer tasks by leveraging Teams SSO and accessing cloud resources down to single line statements with zero configuration. 

With Teams Toolkit you can write user authentication code in a simplified way using Credentials:
* User identity in browser environment: `TeamsUserCredential` represents Teams current user's identity.
* User identity in Node.js environment: `OnBehalfOfUserCredential` uses On-Behalf-Of flow and Teams SSO token.
* Application Identity in Node.js environment: `AppCredential` represents the application identity.

To learn more read the [documentation](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/teamsfx-sdk) or check out the API [reference](https://docs.microsoft.com/en-us/javascript/api/@microsoft/teamsfx/?view=msteams-client-js-latest).

You can also explore more samples with SSO built by Teams Framework in the [repo](https://github.com/OfficeDev/TeamsFx-Samples/tree/v2)

<p align="right"><a href="#Add-single-sign-on">back to top</a></p>
