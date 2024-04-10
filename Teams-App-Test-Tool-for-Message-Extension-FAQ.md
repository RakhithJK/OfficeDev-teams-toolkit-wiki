## The "Command ID" and "Parameter name" parameters in search command

![image](https://github.com/OfficeDev/TeamsFx/assets/9698542/3e04557d-3805-46f8-b6d2-f5d80b2e992e)

In Teams, when you search in a search command within a message extension app, your app will receive a invoke activity with these two parameters. In rare cases, you app could use these two pieces of information to determine different behaviors of search command (e.g. return different types of search results). **But most of the time, you app doesn't need it** because [Teams only support a single search command](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#composeextensionscommands).

Teams retrieves this information from your app's manifest, but Teams App Test Tool doesn't process the manifest (see [Limitations](https://aka.ms/teams-app-test-tool-manifest-not-processed)). So if your app requires these two parameters (e.g. runs different logic based on `comamnd ID` or `parameter name`), you can set them in the form by clicking `Specify Command ID or Parameter`. 

## The "Command ID" parameter in action command