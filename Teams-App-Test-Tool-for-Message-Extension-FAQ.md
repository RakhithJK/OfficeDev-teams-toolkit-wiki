## The "Command ID" and "Parameter name" Parameters in Search Command

<img height=400px src='https://github.com/OfficeDev/TeamsFx/assets/9698542/3e04557d-3805-46f8-b6d2-f5d80b2e992e'/>

In Teams, when you type in search box within a search-based message extension app, your app will receive an invoke activity with these two parameters. In rare cases, you app could use `activity.value.commandId` or `activity.value.parameters[0].name` to control different behaviors of search command in the activity handler for the `composeExtension/query` invoke activty (e.g. `handleTeamsMessagingExtensionQuery` method in `botbuilder-js` SDK). **But most of the time, you app doesn't need it** because [Teams only support a single search command](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#composeextensionscommands) and you can leave them empty.

In case your app reads these two parameters, you can expand the inputs by clicking `Specify Command ID or Parameter` and fill in the values. When searching, Teams App Test Tool will send these two parameters in the invoke activity payload. If not set, the activity payload won't contain them.

> In Teams, this information is retrieved from your app's manifest. However, Teams App Test Tool doesn't process the manifest (see [Limitations](https://aka.ms/teams-app-test-tool-manifest-not-processed)), so you need to manually specify it.

For details about the schema of this invoke activity payload, please refer to [Teams documentation](https://learn.microsoft.com/en-us/microsoftteams/platform/messaging-extensions/how-to/search-commands/respond-to-search?tabs=json#respond-to-user-requests)

## The "Command ID" Parameter in Action Command

<img height=400px src='https://github.com/OfficeDev/TeamsFx/assets/9698542/44935432-cc11-4da2-a20f-2b6de9e16285' />

In Teams, throughout the process of stepping through the dialogs triggered from action commands, your app will receive an `composeExtension/fetchTask` or `composeExtension/submitAction` invoke activity that contains the `activity.value.commandId` parameter. Usually, you app uses it to dispatch different commands in the activity handler for the `composeExtension/fetchTask` or `composeExtension/submitAction` invoke activty (e.g. `handleTeamsMessagingExtensionFetchTask` or `handleTeamsMessagingExtensionSubmitAction` method in `botbuilder-js` SDK).

When testing different action commands, you need to specify it in the `Command ID` input box. If not set, the activity payload won't contain them.

> In Teams, this information is retrieved from your app's manifest. However, Teams App Test Tool doesn't process the manifest (see [Limitations](https://aka.ms/teams-app-test-tool-manifest-not-processed)), so you need to manually specify it.

For details about the schema of this invoke activity payload, please refer to [Create and send dialogs](https://learn.microsoft.com/en-us/microsoftteams/platform/messaging-extensions/how-to/action-commands/create-task-module?tabs=dotnet) and [Respond to the dialog submit action](https://learn.microsoft.com/en-us/microsoftteams/platform/messaging-extensions/how-to/action-commands/respond-to-task-module-submit?tabs=dotnet%2Cdotnet-1).

## Test Static List of Parameters in Action Command

<img height=400px src='https://github.com/OfficeDev/TeamsFx/assets/9698542/db6d1843-9c29-4577-a73c-a9052d639053' />

Static list of parameters is the simplest method to create dialog for action commands, however you can't control the formatting in this case.

If you select to create the dialog with a static list of parameters and when the user submits the dialog, the message extension app will receive an `composeExtension/submitAction` invoke activity.

> Normally, you can define a list of parameters in your app manifest. However, Teams App Test Tool doesn't process the manifest (see [Limitations](https://aka.ms/teams-app-test-tool-manifest-not-processed)), so you need to manually specify it.

For the schema of the static list of parameters, please refer to [Manifest Schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#composeextensionscommands).
