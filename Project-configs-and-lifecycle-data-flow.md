## Config files introduction

### Project settings (configs/projectSettings.json)
Project settings are environment awareless, it defines some static features of the project, such as capabilities, resources descriptions.
Here is an example:
```
{
    "appName": "tab123",
    "projectId": "1e257247-b474-4a7b-9239-1bfb58b2379c",
    "solutionSettings": {
        "name": "fx-solution-azure",
        "version": "1.0.0",
        "hostType": "Azure",
        "capabilities": [
            "Tab"
        ],
        "azureResources": [],
        "activeResourcePlugins": [
            "fx-resource-frontend-hosting",
            "fx-resource-aad-app-for-teams",
            "fx-resource-local-debug",
            "fx-resource-appstudio",
            "fx-resource-simple-auth"
        ]
    },
    "programmingLanguage": "javascript"
}
```
Project settings file is created by core when user creates a new project. 
Core will fill in basic parts of project settings for example: `appName` and `projectId`. Once created, the basic information will never be changed and will be shared across all plugins in all environments.

Solution will fill in the solution part of settings such as `hostType`, `capabilities`, `azureResources`, etc. After creation of project, solution can add more capabilities or azure resources when user trigger the corresponding commands. 

After user create new project or add capability/resource, the project settings file will eventually be persisted on file system by core module.

### Provision input configs (configs/config.dev.json)

Provision input configs are customized config values defined by users, which will be part of provision inputs. Here is an example:
```
{
    "$schema": "https://raw.githubusercontent.com/OfficeDev/TeamsFx/dev/packages/api/src/schemas/envConfig.json",
    "azure": {
      "subscriptionId": "123-456-789",
      "resourceGroupName": "myrg"
    },
    "bot": {
      "appId": "123-456-32",
      "appPassword": "swerwr234"
    },
    "manifest": {
        "description": "You can customize the 'values' object to customize Teams app manifest for different environments. Visit https://aka.ms/teamsfx-config to learn more about this.",
        "values": {
          "appName": {
            "short": "myApp123",
            "full": "full name of my app 123"
          }
        }
    }
}
```
When provisioning resorce, user can define their subscription id and resource group by their own.

When user create a new project, the default environment will be created. 
User can also create more environments instead of the default one. 

Once a new environment is created, a new inital config file will be created with empty customized values.

Once this file is created, toolkit will never modify it. Because it is an input file that only user can modify.

### ARM related input files (azure.parameters.dev.json, bicep tempalets)

ARM related files are another kind of provision input files targeting provisioning resource using arm templates.

ARM related files are scaffolded by Azure soltion, which is invisible to core module. 

Solution plugin is resposible to collect bicep templates from resource plugins and merge the templates and persit on file system.

Solution will execute Azure resource provision task for resources that can be created by ARM templates. For such resources, resource plugin only need to define bicep templates and will never call Azure SDK to provision the resource by their own. 
But there are still some exceptions cases that ARM templates can not cover, for example:
    - Microsoft Entra app registration is not supported by ARM templates.
    - Some data plane operations that are closely bound in provision tasks, for example, create admin user in Azure SQL database.

### Provision output configs (publishProfiles/profile.dev.json)

Provision output configs contains all provision output values of all plugins. This file is used as input for code deployment and application publish.

Provision output configs will contain placeholders for user secret data, whose values are seperated out in a user data file. 

In priciple, the user data file will not be checked in version control systems.

Here is an example:
```
{
    "solution": {
        "resourceNameSuffix": "3166c6",
        "resourceGroupName": "qinezh0831",
        "tenantId": "7309892c-5bd2-4c8e-8a04-7363aea4ac37",
        "subscriptionId": "e6cf7237-8d42-4aeb-b227-a1f01a37883c",
        "location": "East US",
        "teamsAppTenantId": "{{solution.teamsAppTenantId}}",
        "remoteTeamsAppId": "1826ea78-7a3b-4e18-a35d-e4b64ce7b7cc",
        "resource_base_name": "qinezh08313166c6",
        "provisionSucceeded": true
    },
    "fx-resource-appstudio": {},
    "fx-resource-frontend-hosting": {
        "storageName": "frontendstgryhs224ny6ja2",
        "endpoint": "https://frontendstgryhs224ny6ja2.z13.web.core.windows.net",
        "domain": "frontendstgryhs224ny6ja2.z13.web.core.windows.net"
    },
    "fx-resource-aad-app-for-teams": {
        "clientId": "1a6f9bea-6b93-4a06-9303-a49be5b585af",
        "clientSecret": "{{fx-resource-aad-app-for-teams.clientSecret}}",
        "objectId": "8a354c3c-8453-4d25-b279-79386a2b01e5",
        "oauth2PermissionScopeId": "c60068cf-81dd-4d8f-aeeb-ceb043fd206d",
        "tenantId": "7309892c-5bd2-4c8e-8a04-7363aea4ac37",
        "oauthHost": "https://login.microsoftonline.com",
        "oauthAuthority": "https://login.microsoftonline.com/7309892c-5bd2-4c8e-8a04-7363aea4ac37",
        "applicationIdUris": "api://frontendstgryhs224ny6ja2.z13.web.core.windows.net/1a6f9bea-6b93-4a06-9303-a49be5b585af"
    },
    "fx-resource-simple-auth": {
        "endpoint": "https://qinezh08313166c6-simpleauth-webapp.azurewebsites.net",
        "skuName": "F1"
    }
}
```

User is not expected to modify this file because it is purely output. 

Each time user finish provisioning, this file will be replaced automatically. Modifying it is meanless.

### Local settings (localSettings.json)

Local settings are used for local debug scenario. It defines some local configs for resource running locally, for example, localhost endpoint for tab App.
Here is a example schema:
```
{
    "teamsApp": {
        "tenantId": "",
        "teamsAppId": ""
    },
    "frontend": {
        "browser": "none",
        "https": true,
        "trustDevCert": true,
        "sslCertFile": "",
        "sslKeyFile": "",
        "tabDomain": "",
        "tabEndpoint": ""
    },
    "auth": {
        "clientId": "",
        "clientSecret": "",
        "objectId": "",
        "oauth2PermissionScopeId": "",
        "oauthAuthority": "",
        "oauthHost": "",
        "applicationIdUris": "",
        "simpleAuthFilePath": "",
        "SimpleAuthEnvironmentVariableParams": "",
        "AuthServiceEndpoint": ""
    }
}
```
When user trigger F5 for local debuggin, core will create an empty local settings pass it to plugins, who will create local resources and output values in this settings object.

### App package (templates/appPackage)

App package contains the Teams app manifest and icon resources. 

Before publishing, the placeholders in app manifest template will be replaced by values from provision outputs. 

App package is invisible to core, app studio plugin is responsible to manage these artifacts.

## lifecycle inputs and outputs

### createProject
	
input: 
- Answers from user inpus
	
output
- Project settings
- Provision input config
- Scaffoled source codes (tab, api, etc)
- ARM templates, azure.parameters.dev.json
- App package
	 
### provision

input
- [Optionl] User inputs
- Project settings 
- Provision input configs (config.dev.json)
- ARM templates (bicep template files + azure.parameters.dev.json)

output
- Privion output configs (profile.dev.json)

### configResource:

input
- Project settings
- Provision input configs (config.dev.json)
- Privion output configs (profile.dev.json)

output
    - Privion output configs (profile.dev.json)
	
### provisionLocalResource(localDebug)

input
- Project settings

output
- Local settings 

### configLocalResource

input
- Project settings
- Local settings

output
- Local settings

### deploy

input
- user inputs
- Project settings
- Provision output configs (profile.dev.json)

output
- Provision output configs (profile.dev.json)
	
### publish

input
- user inputs
- Project settings
- Provision input configs (config.dev.json)
- Provision output configs (profile.dev.json)
- App package

output
- Provision output configs (profile.dev.json)
