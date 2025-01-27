## Prerequisite

1. **Important**: Visual Studio version >= 17.10 Preview 3

## Debug with "Multi-Project Launch Profiles" Preview Feature

To use this feature:
1. Go to Tools -> Options -> Preview Features
2. Select 'Enable Multi-Project Launch Profiles'
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/enable-multiple-profiles-feature.png)

### Start the app in Outlook
1. Select `Outlook (browser)` in debug dropdown menu
<br/> ![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/switch-to-outlook.png)
2. Press F5, or select Debug > Start Debugging menu in Visual Studio
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/debug-button.png)

### Start the app in Teams App Test Tool

> Note: if your template is `AI Assistant Bot` you should fill in both OpenAI API Key and the created Assistant ID into `appsettings.TestTool.json`
>   ```
>   "OpenAI": {
>      "ApiKey": "<your-openai-api-key>",
>      "AssistantId": "<your-openai-assistant-id>"
>   }
>   ```

> If your template is `AI Chat Bot`, you should fill in your OpenAI API Key or Azure OpenAI settings in `appsettings.TestTool.json`
>    ```
>    # If using OpenAI
>    "OpenAI": {
>      "ApiKey": "<your-openai-api-key>"
>    },
>    # If using Azure OpenAI
>    "Azure": {
>      "OpenAIApiKey": "<your-azure-openai-api-key>",
>      "OpenAIEndpoint": "<your-azure-openai-endpoint>"
>    }
>    ```
> If using Azure OpenAI, update "gpt-35-turbo" in `Program.cs` to your own model deployment name

1. Select `Teams App Test Tool (browser)` in the debug dropdown menu
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/switch-to-test-tool.png)
2. Press F5, or select Debug > Start Debugging menu in Visual Studio
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/debug-button.png)

### Others

Same as above, select the pre-defined profile from the debug dropdown menu.

## Debug without "Multi-Project Launch Profiles" Preview Feature

### Start the app in Outlook
1. Select `TeamsApp` in the project selection dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/e581920f-194e-4e6b-93c2-22ad633e328e)
2. Select `Outlook (browser)` in the debug dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/d32a8f6b-901d-4e0f-a612-9deceea8b196)
3. Select `Configure Startup Projects...` in the debug dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/2dffb4e4-492e-43bc-bb37-e30e5681600c)
4. In the popup dialog, select `Multiple startup projects` and configure the `Action` as `Start`. Then click "Apply" and "OK".
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/a195a824-45ac-45fc-92e8-3ebd9c582cc9)
5. Press F5, or select Debug > Start Debugging menu in Visual Studio
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/debug-button.png)

### Start the app in Teams App Test Tool

> Note: if your template is `AI Assistant Bot` you should fill in both OpenAI API Key and the created Assistant ID into `appsettings.TestTool.json`
>   ```
>   "OpenAI": {
>      "ApiKey": "<your-openai-api-key>",
>      "AssistantId": "<your-openai-assistant-id>"
>   }
>   ```

> If your template is `AI Chat Bot`, you should fill in your OpenAI API Key or Azure OpenAI settings in `appsettings.TestTool.json`
>    ```
>    # If using OpenAI
>    "OpenAI": {
>      "ApiKey": "<your-openai-api-key>"
>    },
>    # If using Azure OpenAI
>    "Azure": {
>      "OpenAIApiKey": "<your-azure-openai-api-key>",
>      "OpenAIEndpoint": "<your-azure-openai-endpoint>"
>    }
>    ```
> If using Azure OpenAI, update "gpt-35-turbo" in `Program.cs` to your own model deployment name

1. Select `TeamsApp` in the project selection dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/e581920f-194e-4e6b-93c2-22ad633e328e)
2. Select `Teams App Test Tool (browser)` in the debug dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/d32a8f6b-901d-4e0f-a612-9deceea8b196)
3. Select `{{YOUR_CSHARP_PROJECT}}` in the project selection dropdown menu, such as `MyTeamsApp1`.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/e581920f-194e-4e6b-93c2-22ad633e328e)
4. Select `Teams App Test Tool` in the debug dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/f8f3ff12-ae63-48c7-9cd1-4e6adafd0635)
5. Select `Configure Startup Projects...` in the debug dropdown menu.
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/2dffb4e4-492e-43bc-bb37-e30e5681600c)
6. In the popup dialog, select `Multiple startup projects` and configure the `Action` as `Start`. Then click "Apply" and "OK".
<br/>![image](https://github.com/OfficeDev/TeamsFx/assets/15262146/a195a824-45ac-45fc-92e8-3ebd9c582cc9)
7. Press F5, or select Debug > Start Debugging menu in Visual Studio
<br/>![image](https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/docs/images/visualstudio/debug/debug-button.png)

### Others

Same as above, select the correct profile of the `TeamsApp` and `{{YOUR_CSHARP_PROJECT}}`, configure `Multiple startup projects` and then start it.
