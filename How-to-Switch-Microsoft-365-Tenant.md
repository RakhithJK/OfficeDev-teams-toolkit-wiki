This doc is to help you mitigate the error when the Microsoft 365 tenant of your currently signed-in account does not match with what you previously used. 

# Why
The error may occur when you local debug or kick off provisioning resources in a remote environment but we notice that the Microsoft 365 tenant you are currently using is different from what recorded in .env file. We will not provision AAD or Bot resources in the new tenant by default but would like to ask you to confirm the account and then follow the mitigation steps mentioned below to either fix the wrong account or continue provisioning resources in the new tenant.


# Mitigation
1. Check your Microsoft 365 account.    
    a) If you switched to the account unintentionally , please sign out of the current account and sign in with the correct one. Continue local debugging or provision in remote environemnt.     
    b) If you plan to continue with the new account to provision resources in new tenant, please follow step 2.    
2. To provision resources in new tenant, 
    - Clear the value of following keys in `.env.{env}` file in teamsfx folder. For example, the file would be .env.dev for dev environment,
        -  Clear the value of TEAMS_APP_TENANT_ID in .env.
        - Clear the value of AAD_APP_CLIENT_ID if you need an AAD aap.
        - Clear the value of BOT_ID if your project includes a Bot app.
    - Start local debugging or provision, and Teams Toolkit will provision resources in the new Microsoft 365 tenant.



## Troubleshoot
### Could not be Redirected to the Expected Teams Web Page
If you have previewed (local or remote) your Teams app in one Microsoft 365 tenant and then switch to another Microsoft 365 account, you may encounter error as shown below 
![teams-signin-error](https://github.com/OfficeDev/TeamsFx/assets/86260893/448b82c6-b785-4871-b2c9-9ce17750389f)

once the browser is launched when previewing in the new Microsoft 365 tenant. If clicking "try again" or waiting for a few seconds to let Teams bring you to the sign in page, you may notice that the page won't be redirected correctly to the page of adding the Teams app. This happens due to the previous account info saved in the browser storage.

#### Mitigation
* Launch browser with userData    
By default, the browser is launched with a separate user profile in a temp folder. You could override the value of "userDataDir" to "true" and then specify the path of user data folder in runtimeArgs.
  *  Visual Studio Code    
  For example, when you sign in with another Microsoft 365 account for local debugging, you could replace    
      ```
      {
        "name": "Attach to Frontend (Edge)",
          "type": "pwa-msedge",
          "request": "launch",
          "url": "https://teams.microsoft.com/l/app/${localTeamsAppId}?installAppPackage=true&webjoin=true&${account-hint}",
          "presentation": {
              "group": "all",
              "hidden": true
          }
      }
      ```
      with 
      ```
      {
        "name": "Attach to Frontend (Edge)",
          "type": "pwa-msedge",
          "request": "launch",
          "url": "https://teams.microsoft.com/l/app/${localTeamsAppId}?installAppPackage=true&webjoin=true&${account-hint}",
          "presentation": {
              "group": "all",
              "hidden": true
          },
          "userDataDir": true, // Enable to use customized user data folder.
          "runtimeArgs": [
            "--user-data-dir=C:\\Users\\{username}\\temp\\edge\\tenantb" // Pass the path of user data folder here.
          ]
      }
      ```
      If you want to switch back to the previous Microsoft 365 tenant for local debugging, please remove the lines about userDataDir and runtimeArgs that you just added before starting local debugging again.

      You could also specify the path of user data folder for each tenant, and edit the value of "user-data-dir" in runtimeArgs whenever you switch tenant for preview.

  * Visual Studio    
  When running local debug of a Teams project launched in Visual Studio, you could create a new browser configuration after switching to another Microsoft 365 tenant by following steps mentioned in [Add Browser Configuration in Visual Studio](#add-browser-configuration-in-visual-studio). Type `--user-data-dir=C:\\Users\\{username}\\temp\\edge\\tenantb` (replace the path with what it makes sense to you) as the argument when adding the program. And then choose the corresponding browser configuration before local debugging.    
  
     If you want to preview a Teams app in Visual Studio after switching Microsoft 365 tenant, you could copy the preview URL shown in the output pane and then run your browser with arguments using command line. For example, you could start Edge with `msedge.exe --user-data-dir="C:\\Users\\{username}\\temp\\edge\\tenantb"`. Once the browser is launched, paste the preview URL.
 
* Launch browser in incognito mode    
  This may not work for you if your org enables condition access. 
  * Visual Studio Code     
  runtimeArgs are the arguments passed to the runtime executable. You could edit the launch configuration by adding `"runtimeArgs": ["--inprivate"]` (for Edge) or `"runtimeArgs": ["--incognito"]` (for Chrome) to launch the browser in incognito mode. For example, you could replace 
    ```
    {
       "name": "Attach to Frontend (Edge)",
        "type": "pwa-msedge",
        "request": "launch",
        "url": "https://teams.microsoft.com/l/app/${localTeamsAppId}?installAppPackage=true&webjoin=true&${account-hint}",
        "presentation": {
            "group": "all",
            "hidden": true
        }
    }
    ```
    with 
    ```
    {
       "name": "Attach to Frontend (Edge)",
        "type": "pwa-msedge",
        "request": "launch",
        "url": "https://teams.microsoft.com/l/app/${localTeamsAppId}?installAppPackage=true&webjoin=true&${account-hint}",
        "presentation": {
            "group": "all",
            "hidden": true
        },
        "runtimeArgs": ["--inprivate"] // runtimeArgs that you need to add
    }
    ```

    to always start Edge in InPrivate browsing mode when local debugging.

  * Visual Studio     
    Similarly, for a Teams project launched in Visual Studio, you could create a new browser configuration by following steps mentioned in [Add Browser Configuration in Visual Studio](#add-browser-configuration-in-visual-studio). For arguments when adding the program, type `--inPrivate` (Edge) or `--incognito` (Chrome).

     If you want to preview a Teams app in a remote environment, you could launch the browser in incognito mode and then copy the preview URL shown in the output pane and paste it in the browser.
    
### Could not Authorize or Send Request in Visual Studio
After preparing Teams app dependencies again in Visual Studio with another Microsoft 365 account in a different tenant, you may notice issues like receiving 401 response when sending a bot command or could not authorize to get the user's profile photo in the tab as the image shown below when local debugging.
![vs-authorize-error](https://github.com/OfficeDev/TeamsFx/assets/86260893/cbf87279-8b9b-40b2-a575-85718fbf0f41)

We are still improving this scenario but for now a workaround is:
1. Please keep a copy of the current content of appsettings.Development.json.
2. Delete appsettings.Development.json
3. Run F5 again.
4. If you have customized appsettings.Development.json before, please restore these values based on the backup.

### 409 Conflict error for Teams app creation
You may meet 409 conflict error when the Teams app id provided in `.env.{env}.json` file is conflicting with another Teams app under the same tenant. This usually happens when developers work on the same project, or switch account under same tenant. To resolve it, you can either be added as the owner of existing Teams app, or use another Teams app id to avoid conflict.

#### Teams app owner
You need to know who owns the existing Teams app, and let the owner add your M365 account to the owner list. Please refer to [Collaborate on Teams project using Microsoft Teams Toolkit](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/teamsfx-collaboration).

#### Use another app id
You can manually update Teams app id in `.env.{env}.json` file. Run "Provision to the Cloud" again to create the Teams app. Teams Toolkit will generate a new Teams app id.

### Set Up Bot Error
An error with name "AlreadyCreatedBotNotExist" may pop up when local debugging a bot project while the bot id is provided in `env.local` file. This usually happens when you have local debugged a project with one Microsoft 365 account, and then switched to another account in the same tenant and run local debugging. To resolve it, you can either add the new account as the owner of the existing bot, or create a new bot. 

#### Add Bot Owner
You need to know who owns the existing bot, and visit https://dev.botframework.com/bots with the account owning the bot now. And then you could add owners in the "Settings" page.     
![add-bot-owner](https://github.com/OfficeDev/TeamsFx/assets/86260893/66748bbb-0529-4871-b5da-c38cb1c1ce23)

Please try remove value of bot id in env.local if this does not work for you and Teams Toolkit will create a new bot for you when you start local debugging again or prepare Teams app dependencies if in Visual Studio.

## Appendix 
### Add Browser Configuration in Visual Studio
To create a new browser configuration in Visual Studio, you could
1. Open the dropdown and select "Browser with".  
![vs-open-browser-with](https://github.com/OfficeDev/TeamsFx/assets/86260893/501249db-e430-4914-a9d2-f6f940b87809)
2. Select "Ädd" to add a new profile   
![vs-open-browser-with](https://github.com/OfficeDev/TeamsFx/assets/86260893/b9b0c543-c625-4f70-88d0-2f7e92e0fb90)
3. Find the path of the program, type the arguments you need in the field of "Arguments", and give it a friendly name. For example, we add a new configuration for Edge inPrivate mode as shown in the image below.    
![vs-add-browser-program](https://github.com/OfficeDev/TeamsFx/assets/86260893/573e7929-cfb3-4de1-9201-6033cd59b0b6)
4. Select the newly added broswer configuration and then Visual Studio will launch browser with the selected configuration.    
![vs-switch-browser-configuration](https://github.com/OfficeDev/TeamsFx/assets/86260893/3cc37075-df5a-4653-962d-97f770087c0a)
