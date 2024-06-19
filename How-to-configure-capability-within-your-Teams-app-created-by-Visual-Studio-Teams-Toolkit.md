# Configure capability within your Teams app created by Visual Studio Teams Toolkit

## Prerequisites

There is a project named `xxx.ttkproj` and ensure that:

- Visual Studio version is greater than 17.10 Generally Available.
- Teams manifest file is under its `appPackage` folder.
- You have a Microsoft 365 account to test the application. If not, you can visit [Microsoft 365 developer program](https://aka.ms/vsc-ttk-create-m365-dev-account) to create one.

Prior to continuing, we strongly recommend creating and going through an app with related capability by Teams Toolkit. You can find all templates in Visual Studio Teams Toolkit (starting from 17.10 GA).

## Introduction to capability

### Tab

Tabs are Teams-aware webpages embedded in Microsoft Teams. They're simple HTML <iframe\> tags that point to domains declared in the app manifest and can be added as part of a channel inside a team, group chat, or personal app for an individual user. You can include custom tabs with your app to embed your own web content in Teams or add Teams-specific functionality to your web content. Learn more from [Build tabs for Teams
](https://learn.microsoft.com/microsoftteams/platform/tabs/what-are-tabs).

Please go to: [Configure Tab capability](#Configure-Tab-capability)

### Bot

A bot, chatbot, or conversational bot is an app that responds to simple commands sent in chat and replies in meaningful ways. Examples of bots in everyday use include: bots that notify about build failures, bots that provide information about the weather or bus schedules, or provide travel information. A bot interaction can be a quick question and answer, or it can be a complex conversation. Being a cloud application, a bot can provide valuable and secure access to cloud services and corporate resources. Learn more from [Build bots for Teams
](https://learn.microsoft.com/microsoftteams/platform/bots/what-are-bots).

Please go to: [Configure Bot capability](#Configure-Bot-capability)

### Message Extension
Message Extension allows users to interact with your web service while composing messages in the Microsoft Teams client. Users can invoke your web service to assist message composition, from the message compose box, or from the search bar.

Message Extensions are implemented on top of the Bot support architecture within Teams. Learn more from [Build message extensions for Teams
](https://learn.microsoft.com/en-us/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions?tabs=dotnet).

Please go to: [Configure Message Extension capability](#Configure-Message-Extension-capability)

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
    ```
      "configurableTabs": [
          {
              "configurationUrl": "${{TAB_ENDPOINT}}/config",
              "canUpdateConfiguration": true,
              "scopes": [
                  "team",
                  "groupchat"
              ]
          }
      ],
    ```
    Make sure `contentUrl`, `websiteUrl` and `configurationUrl` are right. If your Tab is not configurable, you can ignore `configurableTabs` field.
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

1. Update `teamsapp.local.yml`. Add action `botAadApp/create` and `botFramework/create` under provision. Then update `file/createOrUpdateEnvironmentFile` action under deploy:
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
    Replace `BOT_SOURCE_CODE_PROJECT_PATH` with your Bot source code. `BOT_ID` and `BOT_PASSWORD` is using in the runtime. If you have registered it before, you can just configure `.env.local` to add `BOT_ID` to it.

1. Configure your Bot source code to use `BOT_ID` and `BOT_PASSWORD`. No changes if your app is using them before.

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
                        "compose"
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
                },
                {
                    "id": "shareMessage",
                    "context": [
                        "message"
                    ],
                    "description": "Test command to run action on message context (message sharing)",
                    "title": "Share Message",
                    "type": "action",
                    "parameters": [
                        {
                            "name": "includeImage",
                            "title": "Include Image",
                            "description": "Include image in Hero Card",
                            "inputType": "toggle"
                        }
                    ]
                },
                {
                    "id": "searchQuery",
                    "context": [
                        "compose",
                        "commandBox"
                    ],
                    "description": "Test command to run query",
                    "title": "Search",
                    "type": "query",
                    "parameters": [
                        {
                            "name": "searchQuery",
                            "title": "Search Query",
                            "description": "Your search query",
                            "inputType": "text"
                        }
                    ]
                }
            ],
            "messageHandlers": [
                {
                    "type": "link",
                    "value": {
                        "domains": [
                            "*.botframework.com"
                        ]
                    }
                }
            ]
        }
    ]
    ```

1. Update `teamsapp.local.yml`. Add action `botAadApp/create` and `botFramework/create` under provision. Then update `file/createOrUpdateEnvironmentFile` action under deploy:
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

1. Follow [Prepare for local debugging](#Prepare-for-local-debugging) then start local debugging. You will see a Teams website in a new browser opens and ask you to install your app.

## Prepare for local debugging

1. Right-click `ttkproj` and run `Teams Toolkit -> Prepare Teams App Dependencies`. Select or login your Microsoft 365 account in the pop-up account picker.

1. Configure startup projects to let `ttkproj` and your source code start up together. [More details](https://aka.ms/vs-ttk-debug-multi-profiles).
