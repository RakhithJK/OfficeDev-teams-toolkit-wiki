# Enable Single Sign-on in Teams Toolkit for Visual Studio

You can add Single Sign-on to your Teams Project using the Teams Toolkit for Visual Studio. Click Visual Studio menu Project -> Teams Toolkit -> Add Authentication Code.
![image](https://user-images.githubusercontent.com/113089977/203260466-cc6ab581-c391-425d-8503-53a8f4044c66.png)

Teams Toolkit helps you generate the authentication files in "TeamsFx-Auth" folder, including a manifest template file for Microsoft Entra application and authentication redirect pages. Then you will need to link the files to your Teams application by updating authentication configurations to make sure the Single Sign-on works for your application. Please be noted that for different Teams application type like Tab or Bot, the detailed steps are slightly different.

Basically you will need take care these configurations: 

* In the Microsoft Entra manifest file, you need to specify URIs such as the URI to identify the Microsoft Entra authentication app and the redirect URI for returning token. 
* In the Teams manifest file, add the SSO application to link it with Teams application. 
* Add SSO application information in Teams Toolkit configuration files in order to make sure the authentication app can be registered on backend service and started by Teams Toolkit when you debugging or previewing Teams application.

**Note that now we have two kinds of folder structure. Please kindly choose the placeholders according to your VS version.**
- For VS 17.10 Preview 3 or later:
  * PROJECT_FOLDER: ./{YOUR-APP-NAME}
  * TEAMSAPP_FOLDER: ./TeamsApp
- Others:
  * PROJECT_FOLDER: ./
  * TEAMSAPP_FOLDER: ./

For Teams Tab Application
-------------------------
1. Update Microsoft Entra app manifest
`{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/aad.manifest.template.json` is an Microsoft Entra manifest template. You can copy and paste this file to any folder of your project, rename as `aad.manifest.json` and take notes of the path to this file. Because the path will be useful later. And you need to make the following updates in the template to create/update an Microsoft Entra app for SSO:

    1. "identifierUris": Used to uniquely identify and access the resource.
    [HelpLink.](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest#identifieruris-attribute)
    You need to set correct Redirect Uris into "identifierUris" for successfully identify this app.

    Example for TeamsFx Tab template
    ```
    "identifierUris":[
      "api://${{TAB_DOMAIN}}/${{AAD_APP_CLIENT_ID}}"
    ]
    ```
 
    2. "replyUrlsWithType": List of registered redirect_uri values that Microsoft Entra will accept as destinations when returning tokens.
    [HelpLink.](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest#replyurlswithtype-attribute)
    You need to set necessary Redirect Uris into "replyUrlsWithType" for successfully returning token.
    For example:
    ```
    "replyUrlsWithType":[
      {
        "url": "${{TAB_ENDPOINT}}/auth-end.html",
        "type": "Web"
      }
    ]
    ```
    > Note: You can use `${{ENV_NAME}}` to reference variables in `{{TEAMSAPP_FOLDER}}/env/.env.{TEAMSFX_ENV}`.

    Example for TeamsFx Tab template
    ```
    "replyUrlsWithType":[
      {
        "url": "${{TAB_ENDPOINT}}/auth-end.html",
        "type": "Web"
      },
      {
        "url": "${{TAB_ENDPOINT}}/auth-end.html?clientId=${{AAD_APP_CLIENT_ID}}",
        "type": "Spa"
      },
      {
        "url": "${{TAB_ENDPOINT}}/blank-auth-end.html",
        "type": "Spa"
      }
    ]
    ```
   3. "name": Replace the value with your expected Microsoft Entra app name.

2. Update Teams app manifest. Open your Teams app manifest file, update the content of the `validDomain` property and add a `WebApplicationInfo` object with the value of your SSO app.[HelpLink.](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#webapplicationinfo)
  
    For example:
    ```
    "validDomains": [
      "${{TAB_HOSTNAME}}"
    ],
    "webApplicationInfo": {
      "id": "${{AAD_APP_CLIENT_ID}}",
      "resource": "SAME_AS_YOUR_IDENTIFIERURIS"
    }
    ```
    > Note: update the value of resource to your `identifierUris` configed in step 1.i, and use `${{ENV_NAME}}` to reference envs in `{{TEAMSAPP_FOLDER}}/env/.env.{TEAMSFX_ENV}`.

    Example for TeamsFx Tab template

    Open `{{TEAMSAPP_FOLDER}}/appPackage/manifest.json`, update the content of the `validDomain` property:
    ```
    "validDomains": [
      "${{TAB_HOSTNAME}}"
    ]
    ```

    Then, append the following object in the manifest:
    ```
    "webApplicationInfo": {
      "id": "${{AAD_APP_CLIENT_ID}}",
      "resource": "api://${{TAB_DOMAIN}}/${{AAD_APP_CLIENT_ID}}"
    }
    ```
 
3. Update `{{TEAMSAPP_FOLDER}}/teamsapp.yml` and `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml`
  Microsoft Entra related changes and configs needs to be added into your `yml` files:
    - add `aadApp/create` under `provision`:
      For creating new Microsoft Entra apps used for SSO.
      [HelpLink](https://aka.ms/teamsfx-actions/aadapp-create)
    - add `aadApp/update` under `provision`
      For updating your Microsoft Entra app with Microsoft Entra app manifest in step 1.
      [HelpLink](https://aka.ms/teamsfx-actions/aadapp-update)
    - update `file/createOrUpdateJsonFile`
      For adding following environment variables when local debug:
        a. ClientId: Microsoft Entra app client id
        b. ClientSecret: Microsoft Entra app client secret
        c. OAuthAuthority: Microsoft Entra app oauth authority
      [HelpLink](https://github.com/OfficeDev/TeamsFx/wiki/Available-actions-in-Teams-Toolkit#fileupdatejson)


    Example for TeamsFx Tab template
    
    In both `{{TEAMSAPP_FOLDER}}/teamsapp.yml` and `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml` files:
    - Add following lines under `provision` to create Microsoft Entra app.
      ```
      - uses: aadApp/create
        with:
          name: "YOUR_AAD_APP_NAME"
          generateClientSecret: true
          signInAudience: "AzureADMyOrg"
        writeToEnvironmentFile:
          clientId: AAD_APP_CLIENT_ID
          clientSecret: SECRET_AAD_APP_CLIENT_SECRET
          objectId: AAD_APP_OBJECT_ID
          tenantId: AAD_APP_TENANT_ID
          authority: AAD_APP_OAUTH_AUTHORITY
          authorityHost: AAD_APP_OAUTH_AUTHORITY_HOST
      ```
      > Note: Replace the value of "name" with your expected Microsoft Entra app name.
    
    - Add following lines under `provision` to configure Microsoft Entra app with Microsoft Entra app template in the step 1.
      ```
      - uses: aadApp/update
        with:
          manifestPath: "YOUR_PATH_TO_AAD_APP_MANIFEST"
          outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json
      ```
      > Note: Replace the value of `manifestPath` with the relative path of Microsoft Entra app manifest noted in step 1. For example, `./aad.manifest.json`

    In `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml` only:
    - Add following lines under `provision` to add Microsoft Entra related configs to local debug service.
      ```
        - uses: file/createOrUpdateJsonFile
          with:
            target: ./{YOUR-APP-NAME}/appsettings.Development.json
            appsettings:
              TeamsFx:
                Authentication:
                  ClientId: ${{AAD_APP_CLIENT_ID}}
                  ClientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
                  InitiateLoginEndpoint: ${{TAB_ENDPOINT}}/auth-start.html
                  OAuthAuthority: ${{AAD_APP_OAUTH_AUTHORITY}}
      ```
    - Update `script` under `provison` to add env variables for your tab project.
      ```
      - uses: script
        with:
          run:
            echo "::set-teamsfx-env TAB_HOSTNAME=localhost";
            echo "::set-teamsfx-env TAB_DOMAIN=localhost:44302";
            echo "::set-teamsfx-env TAB_ENDPOINT=https://localhost:44302";
      ```

4. Update Infra
   Microsoft Entra related configs needs to be configured in your remote service. Following example shows the configs on Azure Webapp.
     1. TeamsFx__Authentication__ClientId: Microsoft Entra app client id
     2. TeamsFx__Authentication__ClientSecret: Microsoft Entra app client secret
     3. TeamsFx__Authentication__OAuthAuthority: Microsoft Entra app oauth authority
  
   Example for TeamsFx Tab template

   Open `{{TEAMSAPP_FOLDER}}/infra/azure.parameters.json` and add following lines into `parameters`:
   ```
   "tabAadAppClientId": {
     "value": "${{AAD_APP_CLIENT_ID}}"
   },
   "tabAadAppClientSecret": {
     "value": "${{SECRET_AAD_APP_CLIENT_SECRET}}"
   },
   "tabAadAppOauthAuthorityHost": {
     "value": "${{AAD_APP_OAUTH_AUTHORITY_HOST}}"
   },
   "tabAadAppTenantId": {
     "value": "${{AAD_APP_TENANT_ID}}"
   }
   ```

   Open `{{TEAMSAPP_FOLDER}}/infra/azure.bicep` find follow line:
   ```
   param location string = resourceGroup().location
   ```
   and add following lines:
   ```
   param tabAadAppClientId string
   param tabAadAppOauthAuthorityHost string
   param tabAadAppTenantId string
   @secure()
   param tabAadAppClientSecret string
   ```
   In the same file, update
   ```
   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
      kind: 'app'
      location: location
      name: webAppName
      properties: {
        serverFarmId: serverfarm.id
        httpsOnly: true
        siteConfig: {
          appSettings: [
            {
              name: 'WEBSITE_RUN_FROM_PACKAGE'
              value: '1'
            }
          ]
          ftpsState: 'FtpsOnly'
        }
      }
    }
   ```
   with:
   ```
   resource webApp 'Microsoft.Web/sites@2021-02-01' = {
      kind: 'app'
      location: location
      name: webAppName
      properties: {
        serverFarmId: serverfarm.id
        httpsOnly: true
        siteConfig: {
          ftpsState: 'FtpsOnly'
        }
      }
    }

    resource  webAppConfig  'Microsoft.Web/sites/config@2021-02-01' = {
      name: '${webAppName}/appsettings'
      properties: {
        WEBSITE_RUN_FROM_PACKAGE: '1'
        TeamsFx__Authentication__ClientId: tabAadAppClientId
        TeamsFx__Authentication__ClientSecret: tabAadAppClientSecret
        TeamsFx__Authentication__InitiateLoginEndpoint: 'https://${webApp.properties.defaultHostName}/auth-start.html'
        TeamsFx__Authentication__OAuthAuthority: uri(tabAadAppOauthAuthorityHost, tabAadAppTenantId)
      }
    }
   ```

   Also, append the following content in the same file:
   ```
   output TAB_HOSTNAME string = webApp.properties.defaultHostName
   ```

5. Update `{{PROJECT_FOLDER}}/appsettings.json` and `{{PROJECT_FOLDER}}/appsettings.Development.json`
  Microsoft Entra related configs needs to be configure to your .Net project settings:
    ```
    TeamsFx: {
      Authentication: {
        ClientId: Microsoft Entra app client id
        ClientSecret: Microsoft Entra app client secret,
        InitiateLoginEndpoint: Login Endpoint,
        OAuthAuthority: Microsoft Entra app oauth authority
      }
    }
    ```
   > Note: You can use use `$ENV_NAME$` to reference envs in local/remote service.

   Example for TeamsFx Tab template
  
   Open `{{PROJECT_FOLDER}}/appsettings.json` and `{{PROJECT_FOLDER}}/appsettings.Development.json`, and append the following lines:
   ```
   "TeamsFx": {	
     "Authentication": {	
       "ClientId": "$clientId$",	
       "ClientSecret": "$client-secret$",
       "InitiateLoginEndpoint": "$TAB_ENDPOINT$/auth-start.html",
       "OAuthAuthority": "$oauthAuthority$"
     }	
   }
   ```

6. Update source code. With all changes above, your environment is ready and can update your code to add SSO to your Teams app.
   You can find samples in following pages:
    - TeamsFx SDK: https://www.nuget.org/packages/Microsoft.TeamsFx/
    - Sample Code: under `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/Tab`
  
   Example for TeamsFx Tab template

   1. Create `{{PROJECT_FOLDER}}/Config.cs` and paste the following code:
    ```
    using Microsoft.TeamsFx.Configuration;

    namespace {{YOUR_NAMESPACE}}
    {
        public class ConfigOptions
        {
            public TeamsFxOptions TeamsFx { get; set; }
        }
        public class TeamsFxOptions
        {
            public AuthenticationOptions Authentication { get; set; }
        }
    }
    ```
     > Note: You need to replace `{{YOUR_NAMESPACE}}` with your namespace name
  
   2. Move `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/Tab/GetUserProfile.razor` to `{{PROJECT_FOLDER}}/Components/Pages`
   3. Add the `GetUserProfile` component to your razor page, for example find following line in `{{PROJECT_FOLDER}}/Components/Pages/Hello.razor`:
    ```
    <h1>Hello, World</h1>
    <p>Your app is running @(GetHubName())</p>
    ```
   
   and add following line after it:
    ```
    <GetUserProfile />
    ```

   4. Open `{{PROJECT_FOLDER}}/Program.cs`, find the following line:
    ```
    builder.Services.AddScoped<MicrosoftTeams>();
    ```
    and add following code after:
    ```
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Services.AddTeamsFx(config.TeamsFx.Authentication);
    ```

    > [Optional] Note: You may need to exclude the sample code under `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth` to avoid build failure by adding following lines into your `.csproj` file:
   ```
   <ItemGroup>
     <Compile Remove="TeamsFx-Auth/**/*" />
     <None Include="TeamsFx-Auth/**/*" />
     <Content Remove="TeamsFx-Auth/Tab/GetUserProfile.razor"/>
   </ItemGroup>
   ```

   5. Download `auth-start.html` and `auth-end.html` from [GitHub Repo](https://github.com/OfficeDev/TeamsFx/tree/dev/templates/csharp/sso-tab/wwwroot) to `{{PROJECT_FOLDER}}/wwwroot`.

7. To check the SSO app works as expected, run `Local Debug` in Visual Studio. Or run the app in cloud by clicking `Provision in the cloud` and then `Deploy to the cloud` to make the updates taking effects.

For Teams Bot Applications
-------------------------
1. Update Microsoft Entra app manifest. `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/aad.manifest.template.json` is an Microsoft Entra manifest template. You can copy and paste this file to any folder of your project, rename as `aad.manifest.json` and take notes of the path to this file. Because the path will be useful later. And you need to make the following updates in the template to create/update an Microsoft Entra app for SSO:

   1. "identifierUris": Used to uniquely identify and access the resource.[HelpLink.](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest#identifieruris-attribute) You need to set correct Redirect Uris into "identifierUris" for successfully identify this app.
    
    Example for TeamsFx Bot Template:
    ```
    "identifierUris":[
      "api://botid-${{BOT_ID}}"
    ]
    ```
    > Note: You can use use `${{ENV_NAME}}` to reference variables in `{{TEAMSAPP_FOLDER}}/env/.env.{TEAMSFX_ENV}`.

   2. "replyUrlsWithType": List of registered redirect_uri values that Microsoft Entra will accept as destinations when returning tokens.[HelpLink.](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest#replyurlswithtype-attribute) You need to set necessary Redirect Uris into "replyUrlsWithType" for successfully returning token.
    
    For example:
    ```
    "replyUrlsWithType":[
      {
        "url": "https://${{BOT_DOMAIN}}/bot-auth-end.html",
        "type": "Web"
      }
    ]
    ```
    > Note: You can use use `${{ENV_NAME}}` to reference envs in `{{TEAMSAPP_FOLDER}}/env/.env.{TEAMSFX_ENV}`.

    Example for TeamsFx Bot template
    ```
    "replyUrlsWithType":[
      {
      "url": "https://${{BOT_DOMAIN}}/bot-auth-end.html",
      "type": "Web"
      }
    ]
    ```

   3. "name": Replace the value with your expected Microsoft Entra app name.

2. Update Teams app manifest
  
   1. A `WebApplicationInfo` object needs to be added into your Teams app manifest to enable SSO in the Teams app.[HelpLink.](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#webapplicationinfo)
    
    For example: open your Teams app manifest template, and append the following object in the manifest:
    ```
    "webApplicationInfo": {
      "id": "${{AAD_APP_CLIENT_ID}}",
      "resource": "SAME_AS_YOUR_IDENTIFIERURIS"
    }
    ```
    > Note: You need to update the value of resource to your `identifierUris` configed in step 1.i, and use ${{ENV_NAME}} to reference envs in `{{TEAMSAPP_FOLDER}}/env/.env.{TEAMSFX_ENV}`.

    Example for TeamsFx Bot template

    Open `{{TEAMSAPP_FOLDER}}/appPackage/manifest.json`, and append the following object in the manifest:
    ```
    "webApplicationInfo": {
      "id": "${{AAD_APP_CLIENT_ID}}",
      "resource": "api://botid-${{BOT_ID}}"
    }
    ```

    2. You can also register your command under `commands` in `commandLists` of your bot:
    ```
    {
      "title": "YOUR_COMMAND_TITLE",
      "description": "YOUR_COMMAND_DESCRIPTION"
    }
    ```

    Example for TeamsFx Bot template
    ```
    {
      "title": "show",
      "description": "Show user profile using Single Sign On feature"
    }
    ```
    
    Remember to delete the previous 'helloWorld' command since it is not used.

    3. Also add bot domain to `validDomain`:
    ```
    "validDomains": [
      "${{BOT_DOMAIN}}"
    ]
    ```

3. Update `{{TEAMSAPP_FOLDER}}/teamsapp.yml` and `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml` files:
   Microsoft Entra related changes and configs needs to be added into your `yml` files:
    - add `aadApp/create` under `provision`:
      For creating new Microsoft Entra apps used for SSO.
      [HelpLink](https://aka.ms/teamsfx-actions/aadapp-create)
    - add `aadApp/update` under `provision`
      For updating your Microsoft Entra app with Microsoft Entra app manifest in step 1.
      [HelpLink](https://aka.ms/teamsfx-actions/aadapp-update)
    - update `file/createOrUpdateJsonFile`
      For adding following environment variables when local debug:
        a. ClientId: Microsoft Entra app client id
        b. ClientSecret: Microsoft Entra app client secret
        c. OAuthAuthority: Microsoft Entra app oauth authority
      [HelpLink](https://github.com/OfficeDev/TeamsFx/wiki/Available-actions-in-Teams-Toolkit#fileupdatejson)

   Example for TeamsFx Bot template

   In both `{{TEAMSAPP_FOLDER}}/teamsapp.yml` and `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml` files:
    - Add following lines under `provision` to create Microsoft Entra app.
      ```
      - uses: aadApp/create
        with:
          name: "YOUR_AAD_APP_NAME"
          generateClientSecret: true
          signInAudience: "AzureADMyOrg"
        writeToEnvironmentFile:
            clientId: AAD_APP_CLIENT_ID
            clientSecret: SECRET_AAD_APP_CLIENT_SECRET
            objectId: AAD_APP_OBJECT_ID
            tenantId: AAD_APP_TENANT_ID
            authority: AAD_APP_OAUTH_AUTHORITY
            authorityHost: AAD_APP_OAUTH_AUTHORITY_HOST
      ```
      > Note: Replace the value of "name" with your expected Microsoft Entra app name.
    
    - Add following lines under `provision` to configure Microsoft Entra app with Microsoft Entra app template in the step 1.
      ```
      - uses: aadApp/update
        with:
          manifestPath: "./aad.manifest.json"
          outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json
      ```
      > Note: Replace the value of "manifestPath" with the relative path of Microsoft Entra app manifest noted in step 1.
            For example, './aad.manifest.json'

   In `{{TEAMSAPP_FOLDER}}/teamsapp.local.yml` only:
    - Update `file/createOrUpdateJsonFile` under `provision` to add Microsoft Entra related configs to local debug service.
      ```
      # Generate runtime appsettings to JSON file
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../{{PROJECT_FOLDER}}//appsettings.Development.json
          content:
            BOT_TYPE: 'MultiTenant'
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            TeamsFx:
              Authentication:
                ClientId: ${{AAD_APP_CLIENT_ID}}
                ClientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
                OAuthAuthority: ${{AAD_APP_OAUTH_AUTHORITY}}/${{AAD_APP_TENANT_ID}}
                ApplicationIdUri: api://botid-${{BOT_ID}}
                Bot:
                  InitiateLoginEndpoint: https://${{BOT_DOMAIN}}/bot-auth-start
      ```

4. Update Infra. Microsoft Entra related configs needs to be configure to your remote service. Following example shows the configs on Azure Webapp.
    1. TeamsFx__Authentication__ClientId: Microsoft Entra app client id
    2. TeamsFx__Authentication__ClientSecret: Microsoft Entra app client secret
    3. TeamsFx__Authentication__OAuthAuthority: Microsoft Entra app oauth authority
    4. TeamsFx__Authentication__Bot__InitiateLoginEndpoint: Auth start page for Bot
    5. TeamsFx__Authentication__ApplicationIdUri: Microsoft Entra app identify uris

   Example for TeamsFx Bot template

   Open `{{TEAMSAPP_FOLDER}}/infra/azure.parameters.json` and add following lines into `parameters`:
   ```
   "m365ClientId": {
     "value": "${{AAD_APP_CLIENT_ID}}"
   },
   "m365ClientSecret": {
     "value": "${{SECRET_AAD_APP_CLIENT_SECRET}}"
   },
   "m365TenantId": {
     "value": "${{AAD_APP_TENANT_ID}}"
   },
   "m365OauthAuthorityHost": {
     "value": "${{AAD_APP_OAUTH_AUTHORITY_HOST}}"
   }
   ```

   Open `{{TEAMSAPP_FOLDER}}/infra/azure.bicep` find follow line:
   ```
   param location string = resourceGroup().location
   ```
   and add following lines:
   ```
   param m365ClientId string
   param m365TenantId string
   param m365OauthAuthorityHost string
   @secure()
   param m365ClientSecret string
   ```

   Add following lines before output
   ```
   resource webAppSettings 'Microsoft.Web/sites/config@2021-02-01' = {
     name: '${webAppName}/appsettings'
     properties: {
         TeamsFx__Authentication__ClientId: m365ClientId
         TeamsFx__Authentication__ClientSecret: m365ClientSecret
         TeamsFx__Authentication__Bot__InitiateLoginEndpoint: uri('https://${webApp.properties.defaultHostName}', 'bot-auth-start')
         TeamsFx__Authentication__OAuthAuthority: uri(m365OauthAuthorityHost, m365TenantId)
         TeamsFx__Authentication__ApplicationIdUri: 'api://botid-${identity.properties.clientId}'
         BOT_ID: identity.properties.clientId
         BOT_TENANT_ID: identity.properties.tenantId
         BOT_TYPE: 'UserAssignedMsi'
         RUNNING_ON_AZURE: '1'
     }
   }
   ```
   > Note: If you want add additional configs to your Azure Webapp, please add the configs in the webAppSettings.

5. Update `{{PROJECT_FOLDER}}/appsettings.json` and `{{PROJECT_FOLDER}}/appsettings.Development.json`. Microsoft Entra related configs needs to be configure to your .Net project settings:
    ```
    TeamsFx: {
      Authentication: {
        ClientId: Microsoft Entra app client id
        ClientSecret: Microsoft Entra app client secret,
        OAuthAuthority: Microsoft Entra app oauth authority,
        ApplicationIdUri: Microsoft Entra app identify uri,
        Bot: {
          InitiateLoginEndpoint: Auth start page for Bot
        }
      }
    }
    ```
    > Note: You can use use `$ENV_NAME$` to reference envs in local/remote service.

   Example for TeamsFx Bot template

   Open `{{PROJECT_FOLDER}}/appsettings.json` and `{{PROJECT_FOLDER}}/appsettings.Development.json`, and append the following lines:
   ```
   "TeamsFx": {
     "Authentication": {
       "ClientId": "$clientId$",
       "ClientSecret": "$client-secret$",
       "OAuthAuthority": "$oauthAuthority$",
       "ApplicationIdUri": "$applicationIdUri$",
       "Bot": {
         "InitiateLoginEndpoint": "$initiateLoginEndpoint$"
       }
     }
   }
   ```

6. Update source code. With all changes above, your environment is ready and can update your code to add SSO to your Teams app.
  
   You can find samples in following pages:
    - TeamsFx SDK: https://www.nuget.org/packages/Microsoft.TeamsFx/
    - Sample Code: under `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/Bot`

   Example for TeamsFx Bot template
   
   1. Open `{{PROJECT_FOLDER}}/Config.cs` and replace all with following lines:
    ```
    using Microsoft.TeamsFx.Configuration;

    namespace {{YOUR_NAMESPACE}}
    {
        public class TeamsFxOptions
        {
            public AuthenticationOptions Authentication { get; set; }
        }

        public class ConfigOptions
        {
            public string BOT_ID { get; set; }
            public string BOT_PASSWORD { get; set; }
            public string BOT_TYPE { get; set; }
            public string BOT_TENANT_ID { get; set; }
            public TeamsFxOptions TeamsFx { get; set; }
        }
    }
    ```
    > Note: You need to replace {{YOUR_NAMESPACE}} with your namespace name
    > Note: You may need to delete all `remove` items for these files in  your `.csproj` file
  
   2. Move `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/Bot/SSO` and `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth/Bot/Pages` to `{{PROJECT_FOLDER}}/`
     > Note: Remember to replace '{YOUR_NAMESPACE}' with your project namespace.

   3. Open `{{PROJECT_FOLDER}}/Program.cs`, find following line:
    ```
    builder.Services.AddSingleton<BotFrameworkAuthentication, ConfigurationBotFrameworkAuthentication>();
    ```
    and add the following code below:
    ```
    builder.Services.AddRazorPages();

    // Create the Bot Framework Adapter with error handling enabled.                                        
    builder.Services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

    builder.Services.AddSingleton<IStorage, MemoryStorage>();
    // Create the Conversation state. (Used by the Dialog system itself.)
    builder.Services.AddSingleton<ConversationState>();

    // The Dialog that will be run by the bot.
    builder.Services.AddSingleton<SsoDialog>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    builder.Services.AddTransient<IBot, TeamsSsoBot<SsoDialog>>();

    builder.Services.AddOptions<BotAuthenticationOptions>().Configure(options =>
    {
      options.ClientId = config.TeamsFx.Authentication.ClientId;
      options.ClientSecret = config.TeamsFx.Authentication.ClientSecret;
      options.OAuthAuthority = config.TeamsFx.Authentication.OAuthAuthority;
      options.ApplicationIdUri = config.TeamsFx.Authentication.ApplicationIdUri;
      options.InitiateLoginEndpoint = config.TeamsFx.Authentication.Bot.InitiateLoginEndpoint;
    });
    ```
    Find the following lines:
    ```
    builder.Services.AddSingleton<HelloWorldCommandHandler>();
    builder.Services.AddSingleton<GenericCommandHandler>();
    builder.Services.AddSingleton(sp =>
    {
        var options = new ConversationOptions()
        {
            Adapter = sp.GetService<CloudAdapter>(),
            Command = new CommandOptions()
            {
                Commands = new List<ITeamsCommandHandler> { 
                    sp.GetService<HelloWorldCommandHandler>(), 
                    sp.GetService<GenericCommandHandler>() 
                }
            }
        };

        return new ConversationBot(options);
    });
    ```
    and replace with:
    ```
    builder.Services.AddSingleton(sp =>
    {
      var options = new ConversationOptions()
      {
        Adapter = sp.GetService<CloudAdapter>(),
        Command = new CommandOptions()
        {
          Commands = new List<ITeamsCommandHandler> { }
        }
      };

      return new ConversationBot(options);
    });
    ```

    Find and delete the following code:
      ```
      // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
      builder.Services.AddTransient<IBot, TeamsBot>();
      ```
    
    Find the following code:
      ```
      app.UseEndpoints(endpoints =>
      {
        endpoints.MapControllers();
      });
      ```
      and replace with:
      ```
      app.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
        endpoints.MapRazorPages();
      });
      ```
  
    > [Optional] Note: You may need to exclude the sample code under `{{TEAMSAPP_FOLDER}}/TeamsFx-Auth` to avoid build failure by adding following lines into your `.csproj` file:
    ```
    <ItemGroup>
      <Compile Remove="TeamsFx-Auth/**/*" />
      <None Include="TeamsFx-Auth/**/*" />
      <Content Remove="TeamsFx-Auth/Tab/GetUserProfile.razor"/>
    </ItemGroup>
    ```

7. To check the SSO app works as expected, run `Local Debug` in Visual Studio. Or run the app in cloud by clicking `Provision in the cloud` and then `Deploy to the cloud` to make the updates taking effects.
