## Prerequisites

There is a project named `xxx.ttkproj` and ensure that:

- Visual Studio version is greater than 17.10 Generally Available.
- Teams manifest file is under its `appPackage` folder.
- You have a Microsoft 365 account to test the application. If not, you can visit [Microsoft 365 developer program](https://aka.ms/vsc-ttk-create-m365-dev-account) to create one.

Prior to continuing, we strongly recommend creating and going through an app with related capability by Teams Toolkit. You can find all templates in Visual Studio Teams Toolkit (starting from 17.10 GA).

## Choose the Teams capability for your project

Teams offers multiple interfaces, known as Teams Capabilities. Choose the right capability to integrate your application's functions within Teams. 

If you want to embed your web application to provide a browser-like experience within Teams, configure the Teams Tab capability. For exposing your functions or APIs through chat interactions, configure the Teams Bot capability. Additionally, if you need to offer functions or APIs directly within the message composition box, configure the Teams Message Extension capability.

The following are the steps to add Teams capability to your project.

## Configure Tab capability

1. To configure your tab within a group or channel, or personal scope in your Teams application manifest `appPackage/manifest.json`, follow these examples:
    Examples:
    ```
      "staticTabs": [
          {
              "entityId": "index",
              "name": "Personal Tab",
              "contentUrl": "${{TAB_ENDPOINT}}/tab",
              "websiteUrl": "${{TAB_ENDPOINT}}/tab",
              "scopes": [
                  "personal"
              ]
          }
      ],
    ```
  
    Make sure `contentUrl` and `websiteUrl` are right.
    Refer to [Tab schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#configurabletabs) for customizing them.

1. Add your tab domain to the `validDomains` field.
    Example:
    ```
    "validDomains": [
        "${{TAB_DOMAIN}}"
    ],
    ```
    `TAB_ENDPOINT` and `TAB_DOMAIN` are built-in variables of Teams Toolkit. They will be replaced with the true endpoint in runtime based on your current environment(local, dev, etc.).

1. Update the `teamsapp.local.yml` file and add new actions. These actions will enable your tab project to work seamlessly with Teams Toolkit.

    ```yaml
    provision:
      - uses: script # Set TAB_DOMAIN for local launch
        name: Set TAB_DOMAIN and TAB_ENDPOINT for local launch
        with:
          run: 
            echo "::set-output TAB_DOMAIN=localhost:44302"
            echo "::set-output TAB_ENDPOINT=https://localhost:44302"
    ```
    `TAB_DOMAIN` and `TAB_ENDPOINT` should be the application URL when your source code starts up. Maybe they are defined in `launchSettings.json` of your source code.

1. Follow [Prepare for local debugging](#Prepare-for-local-debugging) then start local debugging. You will see a Teams website in a new browser opens and ask you to install your app.

## Configure Bot capability

1. You can configure bot in `appPackage/manifest.json`. You can also refer to [bot schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#bots) if you want to customize.

    Example: 
    ```json
        "bots": [
            {
                "botId": "${{BOT_ID}}",
                "scopes": [
                    "personal",
                    "team",
                    "groupchat"
                ],
                "supportsFiles": false,
                "isNotificationOnly": false,
                "commandLists": [
                    {
                        "scopes": [
                            "personal",
                            "team",
                            "groupchat"
                        ],
                        "commands": [
                            {
                                "title": "welcome",
                                "description": "Resend welcome card of this Bot"
                            },
                            {
                                "title": "learn",
                                "description": "Learn about Adaptive Card and Bot Command"
                            }
                        ]
                    }
                ]
            }
        ]
    ```

1. If you have had registered a Bot app in Teams platform before, you only need to add `BOT_ID` to `.env.local` file. And skip step #3 & step #4. 

1. If you never registered a Bot app yet, follow the step 3&4 to let Teams Toolkit do it for you.
You will need to edit `teamsapp.local.yml` to tell Teams Toolkit the desired actions. Add action `botAadApp/create` and `botFramework/create` under provision module. Then update `file/createOrUpdateEnvironmentFile` action under deploy module like below code piece:
    ```yml
    provision:
      - uses: botAadApp/create
        with:
          # The Microsoft Entra application's display name
          name: bot-${{TEAMSFX_ENV}}
        writeToEnvironmentFile:
          # The Microsoft Entra application's client id created for bot.
          botId: BOT_ID
          # The Microsoft Entra application's client secret created for bot.
          botPassword: SECRET_BOT_PASSWORD 

      # Create or update the bot registration on dev.botframework.com
      - uses: botFramework/create
        with:
          botId: ${{BOT_ID}}
          name: bot
          messagingEndpoint: ${{BOT_ENDPOINT}}/api/messages
          description: ""
          channels:
            - name: msteams

      # Generate runtime appsettings to JSON file
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../BOT_SOURCE_CODE_PROJECT_PATH/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
    ```
    Replace `BOT_SOURCE_CODE_PROJECT_PATH` with your Bot project source code. Note that `BOT_ID` and `BOT_PASSWORD` are using in the runtime. 

1. Configure your Bot source code project to use `BOT_ID` and `BOT_PASSWORD`. 

1. In the debug dropdown menu, select Dev Tunnels > Create A Tunnel (set authentication type to Public) or select an existing public dev tunnel
</br>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/create-devtunnel-button.png)

1. Follow [Prepare for local debugging](#Prepare-for-local-debugging) then start local debugging. You will see a Teams website in a new browser opens and ask you to install your app.

## Configure Message Extension capability

1. You can configure message extension in `appPackage/manifest.json`. You can also refer to [message extension schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#composeextensions) if you want to customize.

    Example:
    ```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    "id": "createCard",
                    "context": [
                      "compose",
                      "message",
                      "commandBox"
                    ],
                    "description": "Command to run action to create a Card from Compose Box",
                    "title": "Create Card",
                    "type": "action",
                    "parameters": [
                        {
                            "name": "title",
                            "title": "Card title",
                            "description": "Title for the card",
                            "inputType": "text"
                        },
                        {
                            "name": "subTitle",
                            "title": "Subtitle",
                            "description": "Subtitle for the card",
                            "inputType": "text"
                        },
                        {
                            "name": "text",
                            "title": "Text",
                            "description": "Text for the card",
                            "inputType": "textarea"
                        }
                    ]
                }
            ]
        }
    ]
    ```
    ```json
    "permissions": [
        "identity",
        "messageTeamMembers"
    ]
    ```

1. If you have had registered a Message Extensions app in Teams platform before, you only need to add `BOT_ID` to `.env.local` file. And skip step #3 & step #4. 

1. If you never registered a Message Extensions app yet, follow the step #3 & #4 to let Teams Toolkit do it for you.
You will need to edit `teamsapp.local.yml` to tell Teams Toolkit the desired actions. Add action `botAadApp/create` and `botFramework/create` under provision module.
    ```yml
    provision:
      - uses: botAadApp/create
        with:
          # The Microsoft Entra application's display name
          name: bot-${{TEAMSFX_ENV}}
        writeToEnvironmentFile:
          # The Microsoft Entra application's client id created for bot.
          botId: BOT_ID
          # The Microsoft Entra application's client secret created for bot.
          botPassword: SECRET_BOT_PASSWORD 

      # Create or update the bot registration on dev.botframework.com
      - uses: botFramework/create
        with:
          botId: ${{BOT_ID}}
          name: bot
          messagingEndpoint: ${{BOT_ENDPOINT}}/api/messages
          description: ""
          channels:
            - name: msteams

      # Generate runtime appsettings to JSON file
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../MESSAGE_EXTENSION_SOURCE_CODE_PROJECT_PATH/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
    ```
   Replace `MESSAGE_EXTENSION_SOURCE_CODE_PROJECT_PATH` with your Message Extensions project source code. `BOT_ID` and `BOT_PASSWORD` are using in the runtime. 

1. Configure your Message Extension source code to use `BOT_ID` and `BOT_PASSWORD`.

1. In the debug dropdown menu, select Dev Tunnels > Create A Tunnel (set authentication type to Public) or select an existing public dev tunnel
</br>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/create-devtunnel-button.png)

1. Follow [Prepare for local debugging](#Prepare-for-local-debugging) then start local debugging. You will see a Teams website in a new browser opens and ask you to install your app.

## Prepare for local debugging

1. Right-click `ttkproj` and run `Teams Toolkit -> Prepare Teams App Dependencies`. Select or login your Microsoft 365 account in the pop-up account picker.

1. Configure startup projects to let `ttkproj` and your source code start up together. [More details](https://aka.ms/vs-ttk-debug-multi-profiles).
