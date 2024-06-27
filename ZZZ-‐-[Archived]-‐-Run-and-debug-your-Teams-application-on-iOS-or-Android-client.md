> [!IMPORTANT]
> Content in this document has been moved to [Teams platform documentation](https://learn.microsoft.com/microsoftteams/platform/toolkit/debug-mobile?tabs=cline%2Cios1%2Cios2). Please do not refer to or update this document.

# Run and debug your Teams application on iOS or Android client.

Please follow the instructions in this tutorial to debug your bot or message extension apps or preview your tab application on mobile devices.

## Steps for testing a tab app on mobile client

1. Run the Teams tab app locally
   - For VSCode:
     - Update `.vscode/task.json`, add `Start local tunnel` task to make the tab app accessible on the mobile client. 
        ```json
        {
            "version": "2.0.0",
            "tasks": [
                {
                    "label": "Start Teams App Locally",
                    "dependsOn": [
                        "Validate prerequisites",
                        "Start local tunnel", // Add this line
                        "Provision",
                        "Deploy",
                        "Start application"
                    ],
                    "dependsOrder": "sequence"
                },
                {
                    // Add this task
                    "label": "Start local tunnel",
                    "type": "teamsfx",
                    "command": "debug-start-local-tunnel",
                    "args": {
                        "type": "dev-tunnel",
                        "ports": [
                            {
                                "portNumber": 53000,
                                "protocol": "https",
                                "access": "private",
                                "writeToEnvironmentFile": {
                                    "endpoint": "TAB_ENDPOINT",
                                    "domain": "TAB_DOMAIN"
                                }
                            }
                        ],
                        "env": "local"
                    },
                    "isBackground": true,
                    "problemMatcher": "$teamsfx-local-tunnel-watch"
                }
            ]
        }
        ```
        > **Note:** When the access of a tab app is set to `private`, it can only be previewed on the mobile client. However, if you want to preview the app on the mobile client and debug it on web clients, you need to set the access level to `public`. It's worth noting that public access raises safety concerns since the tab app can be visited by anyone who knows the app's URL. 
      - Update `teamsapp.local.yml`, **remove** the following action to avoid setting the `TAB_DPMAIN` and `TAB_ENDPOINT` in the env file. 
         ```yaml
         - uses: script
           with:
             run:
               echo "::set-teamsfx-env TAB_DOMAIN=localhost:53000";
               echo "::set-teamsfx-env TAB_ENDPOINT=https://localhost:53000"; 
         ```

      - If you're using React, update `teamsapp.local.yml`, add the configuration `WDS_SOCKET_PORT=0` to activate hot reloading while debugging React after utilizing the tunnel service. 
         ```yaml
         - uses: file/createOrUpdateEnvironmentFile 
           with: 
             target: ./.localConfigs
             envs: 
               BROWSER: none 
               HTTPS: true 
               PORT: 53000 
               SSL_CRT_FILE: ${{SSL_CRT_FILE}} 
               SSL_KEY_FILE: ${{SSL_KEY_FILE}} 
               REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}} 
               REACT_APP_START_LOGIN_PAGE_URL: ${{TAB_ENDPOINT}}/auth-start.html 
               WDS_SOCKET_PORT: 0 
         ```
      - Use the `Run and Debug Activity Panel` in Visual Studio Code and click the `Debug in Teams` green arrow button. 
    - For CLI: 
      - Install [dev tunnel cli](https://aka.ms/teamsfx-install-dev-tunnel).
      - Login with your M365 Account using the command `devtunnel user login`.
      - Start your local tunnel service by running the command `devtunnel host -p 3978 --protocol http --allow-anonymous`.
      - In the `env/.env.local` file, fill in the values for `BOT_DOMAIN` and `BOT_ENDPOINT` with your dev tunnel URL.
          ```
          TAB_DOMAIN=sample-id-3978.devtunnels.ms
          TAB_ENDPOINT=https://sample-id-3978.devtunnels.ms
          ``` 
       - Executing the command `teamsfx provision --env local` in your project directory. 
       - Executing the command `teamsfx deploy --env local` in your project directory. 
       - Executing the command `teamsfx preview --env local` in your project directory. 
1. Open the sideloading URL and install the app in the Teams website as usual. 

   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/ea136692-5188-46c2-b34e-9c591806afa7" width="800"/>

   > **Note:** When the dev tunnel access is set to `private`, the tab app cannot be displayed within an iframe on the web client. It is because its login page is hosted on "login.microsoftonline.com", which has the `X-FRAME-Options` header set to DENY. If you want to preview the app on the mobile client and debug it on web clients, you need to set the access level to `public`. It's worth noting that public access raises safety concerns since the tab app can be visited by anyone who knows the app's URL. 
   > 
   > <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/be8615da-1b63-43a0-ac3d-81a8eecbdaab" width="800"/>
1. Open Teams on your mobile device and click "More" to find the previewing app. 
   
   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/6c98c48b-d893-408d-b980-cc630690e9de" width="300"/>

   > **Note:** If a user has previously debugged the app, it is advisable for them to clear the cache on their mobile device to ensure immediate app synchronization. After clearing the cache, it may take some time for the app to sync. 
   > 
   > On iOS devices, the Teams app data can be cleared by navigating to Settings -> Teams -> Clear App Data. 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614707-8384e9a8-4675-47d0-a065-57377866a33c.png)
   > 
   > For android devices, the Teams app data can be cleared by navigating to Teams->Settings->Data and storage->Clear app data->Clear data 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614768-096f81b8-fc94-4f25-837d-cf97c588a50b.png)

1. If you are accessing the dev tunnel for the first time, you will need to sign in with your M365 account and confirm the anti-phishing page. 

   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/fd22a01e-3d73-469b-a7f2-caeb8159716c" width="300"/>
   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/061963e8-3e20-44c7-903f-20342e776d3b" width="300"/>

   > **Note:** The login process should only be required once per device, and confirmation of the anti-phishing page must be completed after every installation of the app. 

1. Show a mobile friendly tab app. 
   
   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/fcafa66e-6ca3-4144-b989-56497d88c38d" width="300"/>

1. For Android devices, you can use [DevTools](https://learn.microsoft.com/en-us/microsoftteams/platform/tabs/how-to/developer-tools#access-devtools-from-an-android-device) to debug your tab while it is running.

## Steps for debugging a Teams bot app on mobile client

1. Start a Bot App in VSC / CLI 
   Same to the current Teams Toolkit behavior.  
1. Open the sideloading URL and install the app in the Teams website as usual. 
1. Open Teams on your mobile device and click "More" to find the previewing app. 

   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/6c98c48b-d893-408d-b980-cc630690e9de" width="300">

   > **Note:** If a user has previously debugged the bot app and the Team app manifest file is changes, it is advisable for them to clear the cache on their mobile device to ensure immediate app synchronization. After clearing the cache, it may take some time for the app to sync. 
   > 
   > On iOS devices, the Teams app data can be cleared by navigating to Settings -> Teams -> Clear App Data. 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614707-8384e9a8-4675-47d0-a065-57377866a33c.png)
   > 
   > For android devices, the Teams app data can be cleared by navigating to Teams->Settings->Data and storage->Clear app data->Clear data 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614768-096f81b8-fc94-4f25-837d-cf97c588a50b.png)

1. Debug the bot app on your mobile.  
   <img src="https://github.com/OfficeDev/TeamsFx/assets/49138419/1cd8408a-87d5-4fc8-91a3-923a015452c7" width="300">



 

 

 