> [!IMPORTANT]
> Content in this document has been moved to [Teams platform documentation](https://learn.microsoft.com/microsoftteams/platform/toolkit/aad-manifest-customization). Please do not refer to or update this document.

# Microsoft Entra manifest introduction

> This feature is currently under active development. Report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

The [Microsoft Entra manifest](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest) contains a definition of all the attributes of an Microsoft Entra application object in the Microsoft identity platform. 

Teams Toolkit now manages Microsoft Entra application with this manifest file as the source of truth during your Teams application development lifecycles.

In this document, you will learn:
* [How to customize the Microsoft Entra application](#How-to-customize-the-Microsoft-Entra-manifest-template) 
    * [Add an application permission](#Customize-requiredResourceAccess)
    * [Preauthorize a client application](#Customize-preAuthorizedApplications)
    * [Update redirect URL for authentication response](#Customize-redirect-URLs)
* [How to reference values using placeholders in the Microsoft Entra application manifest template](#Placeholders-in-Microsoft-Entra-manifest-template)
    * [Reference configuration files](#Reference-config-file-values-in-Microsoft-Entra-manifest-template)
    * [Reference state files](#Reference-state-file-values-in-Microsoft-Entra-manifest-template)
    * [Reference environment variables](#Reference-environment-variable-in-Microsoft-Entra-manifest-template)
* [How to preview the Microsoft Entra manifest placeholder values with code lens](#How-to-author-and-preview-Microsoft-Entra-manifest-with-code-lens) 
* [How to deploy Microsoft Entra application changes for local development](#How-to-deploy-Microsoft-Entra-application-changes-for-local-environment)
* [How to deploy Microsoft Entra application changes for remote environment](#How-to-deploy-Microsoft-Entra-application-changes-for-remote-environment)
* [How to find the Microsoft Entra application in the Azure Portal](#How-to-view-the-Microsoft-Entra-app-on-the-Azure-portal) 
* [How to use an existing Microsoft Entra application](#How-to-use-an-existing-Microsoft-Entra-app) 
* [Understand Microsoft Entra application in Teams app development lifecycle](#Microsoft-Entra-application-in-Teams-app-development-lifecycle)

## How to customize the Microsoft Entra manifest template

User can customize Microsoft Entra manifest template to update Microsoft Entra application.

1. Open `aad.template.json` in your project
    
    ![image](https://user-images.githubusercontent.com/11220663/167983903-7093ee6b-6378-4e00-936e-1912f742ccfc.png)

1. Update the template directly or [reference values from another file](#Placeholders-in-Microsoft-Entra-manifest-template). Below we have provided several customization scenarios:
    * [Add an application permission](#Customize-requiredResourceAccess)
    * [Preauthorize a client application](#Customize-preAuthorizedApplications)
    * [Update redirect URL for authentication response](#Customize-redirect-URLs)

1. Follow this [instruction](#How-to-deploy-Microsoft-Entra-application-changes-for-local-environment) to deploy your Microsoft Entra application changes for local environment
1. Follow this [instruction](#How-to-deploy-Microsoft-Entra-application-changes-for-remote-environment) to deploy your Microsoft Entra application changes for remote environment.

### Customize requiredResourceAccess

If your Teams app required more permissions to call API with additional permissions, you need to update `requiredResourceAccess` property in the Microsoft Entra manifest template. Here is an example for this property:

```json
"requiredResourceAccess": [
    {
        "resourceAppId": "Microsoft Graph",
        "resourceAccess": [
            {
                "id": "User.Read", // For Microsoft Graph API, you can also use uuid for permission id
                "type": "Scope" // Scope is for delegated permission
            },
            {
                "id": "User.Export.All",
                "type": "Role" // Role is for application permission
            }
        ]
    },
    {
        "resourceAppId": "Office 365 SharePoint Online",
        "resourceAccess": [
            {
                "id": "AllSites.Read",
                "type": "Scope"
            }
        ]
    }
]
```

* `resourceAppId` property is for different APIs, for `Microsoft Graph` and `Office 365 SharePoint Online`, you can input the name directly instead of uuid, and for other APIs, you need to use uuid.
* `resourceAccess.id` property is for different permissions, for `Microsoft Graph` and `Office 365 SharePoint Online`, you can input the permission name directly instead of uuid, and for other APIs, you need to use uuid.
* `resourceAccess.type` property is used for delegated permission or application permission. `Scope` means delegated permission and `Role` means application permission.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

### Customize preAuthorizedApplications

You can use `preAuthorizedApplications` property to authorize a client application indicates that this API trusts the application and users should not be asked to consent when the client calls this exposed API.
Here is an example for this property:

```json
    "preAuthorizedApplications": [
        {
            "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
            "permissionIds": [
                "{{state.fx-resource-aad-app-for-teams.oauth2PermissionScopeId}}"
            ]
        }
        ...
    ]
```

`preAuthorizedApplications.appId` property is used for the application you want to authorize. If you doesn't know the application id but only knows the application name, you can go to Azure Portal following this steps to search the application to find the id:

1. Go to Azure Portal and open [app registrations(https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)
1. Click `All applications` and search the application name
1. If you find the application that you search for, you can click the application and get the application id from the overview page

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

### Customize redirect URLs

Redirect URLs is used when returning authentication responses (tokens) after successfully authenticating. You can customize redirect URLs using property `replyUrlsWithType`, for example, if you want to add `https://www.examples.com/auth-end.html` as redirect URL, you can add it as below:

```json
"replyUrlsWithType": [
    ...
    {
        "url": "https://www.examples.com/auth-end.html",
        "type": "Spa"
    }
]
```

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## Placeholders in Microsoft Entra manifest template

Microsoft Entra manifest template file contains placeholder arguments with `{{...}}` statements. These statements will be replaced at build time for different environments. With placeholder arguments, you can make references to config file, state file and environment variables.

### Reference state file values in Microsoft Entra manifest template

State file is located in `.fx\states\state.xxx.json` (xxx is represent different environment). A typical state file is as below:

```json
{
    "solution": {
        "teamsAppTenantId": "uuid",
        ...
    },
    "fx-resource-aad-app-for-teams": {
        "applicationIdUris": "api://xxx.com/uuid",
        ...
    }
    ...
}
```

If you want to reference `applicationIdUris` value in `fx-resource-aad-app-for-teams` property, you can use this placeholder argument in the Microsoft Entra manifest: `{{state.fx-resource-aad-app-for-teams.applicationIdUris}}`

### Reference config file values in Microsoft Entra manifest template

Config file is located in `.fx\configs\config.xxx.json` (xxx is represent different environment). A typical config file is as below:

```json
{
  "$schema": "https://aka.ms/teamsfx-env-config-schema",
  "description": "description.",
  "manifest": {
    "appName": {
      "short": "app",
      "full": "Full name for app"
    }
  }
}
```

If you want to reference `short` value, you can use this placeholder argument in the Microsoft Entra manifest: `{{config.manifest.appName.short}}`

### Reference environment variable in Microsoft Entra manifest template

Sometimes you may not want to hardcode the values in Microsoft Entra manifest template. For example, when the value is a secret. Microsoft Entra manifest template file supports referencing the values from environment variables. You can use syntax `{{env.YOUR_ENV_VARIABLE_NAME}}` in parameter values to tell the tooling that the value needs to be resolved from current environment variable.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## How to author and preview Microsoft Entra manifest with code lens

Microsoft Entra manifest template file has code lens to help you better review and edit it.

![codelens-overview](https://user-images.githubusercontent.com/11220663/168004906-685f2ed8-984b-4dd7-a491-bd5f136d116c.png)


### Preview Microsoft Entra manifest template file

At the beginning of the Microsoft Entra manifest template file, there is a preview codelens. Click this codelens, it will generate Microsoft Entra manifest based on the environment you selected.

![codelens-preview](https://user-images.githubusercontent.com/11220663/168004969-08536b83-1d47-4832-810e-a772243509de.png)

### Placeholder argument code lens

Placeholder argument has code lens to help you take quick look of the values for local debug and develop environment. If your mouse hover on the placeholder argument, it will show tooltip box for the values of all the environment.

![codelens-placeholder-argument](https://user-images.githubusercontent.com/11220663/168005056-98843db4-d926-4ed6-9cde-3f8e31925ff4.png)

### Required resource access code lens

Different from official [Microsoft Entra manifest schema](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest) that `resourceAppId` and `resourceAccess` id in `requiredResourceAccess` property only support uuid, Microsoft Entra manifest template in Teams Toolkit also support user readable strings for `Microsoft Graph` and `Office 365 SharePoint Online` permissions. If you input uuid, codelens will show user readable strings, otherwise, codelens will show uuid.

![codelens-resource-access](https://user-images.githubusercontent.com/11220663/168005130-1c81c010-836c-47db-a05c-84e402f02387.png)

### Pre-authorized applications code lens

For `preAuthorizedApplications` property, code lens will show the application name for the per-authorized application id.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## How to deploy Microsoft Entra application changes for local environment

1. Click `Preview` code lens in `aad.template.json`

    ![image](https://user-images.githubusercontent.com/11220663/167984920-dd56c97b-588e-4634-846f-d745610114c3.png)

1. Select `local` environment.

    ![image](https://user-images.githubusercontent.com/11220663/167985022-63a946d3-fa60-4c22-ac1c-02a1d916ef0c.png)

1. Click `Deploy Microsoft Entra Manifest` code lens in `aad.local.json`

    ![image](https://user-images.githubusercontent.com/11220663/167985157-302ecae4-3924-483a-8f73-71fea38219a9.png)

1. And now the changes for Microsoft Entra app used in local environment will be deployed.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## How to deploy Microsoft Entra application changes for remote environment

1. Open the command palette and select: `Teams: Deploy Microsoft Entra app manifest`

    ![image](https://user-images.githubusercontent.com/11220663/167999052-3c45e2f1-b79d-47a3-8929-316dd22d31e2.png)

1. Or you can right click on the `aad.template.json` and select `Deploy Microsoft Entra app manifest` from the context menu

    ![image](https://user-images.githubusercontent.com/11220663/167985671-a5ab1ead-f458-4739-ae29-b49495ad5648.png)

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## How to view the Microsoft Entra app on the Azure portal

1. Copy the Microsoft Entra app client id from `state.xxx.json` (xxx is the environment name that you have deployed the Microsoft Entra app) file in the `fx-resource-aad-app-for-teams` property.

    ![get-Microsoft-Entra-client-id](https://user-images.githubusercontent.com/11220663/168005232-003d8778-03a5-46a9-9ce1-2435849b83ea.png)

1. Go to Azure portal and login your M365 account which **must same with the account which created Teams app**.

1. Open [app registrations page](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps), search the Microsoft Entra app using client id which copied from step 1.

    ![search-Microsoft-Entra-on-portal](https://user-images.githubusercontent.com/11220663/168005294-4f9c8a18-1c84-4d4e-897c-9f81f7f124ee.png)


1. Click Microsoft Entra app from search result to view the detailed information.

1. In Microsoft Entra app information page, click `Manifest` menu to view manifest of this app. The schema of the manifest is same as the one in `aad.template.json` file, for more information about manifest, you can refer this [doc](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest).

    ![view-Microsoft-Entra-manifest-on-portal](https://user-images.githubusercontent.com/11220663/168005367-3c5dee09-8576-447b-9475-a3d0c04d831d.png)


1. You can also click other menu to view or configure Microsoft Entra app through portal. 

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## How to view the Microsoft Entra app on the Azure portal for V5
1.  Copy the Microsoft Entra app client id from "aad.manifest"

## How to use an existing Microsoft Entra app

If you want to use existing Microsoft Entra app for your Teams project, you can refer to this [doc](https://github.com/OfficeDev/TeamsFx/wiki/Customize-provision-behaviors#use-an-existing-Microsoft-Entra-app-for-your-teams-app) for more information.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## Microsoft Entra application in Teams app development lifecycle

You would interact with Microsoft Entra application during various stages of your Teams app development lifecycle and here is how Teams Toolkit makes it easy.

### 1. Project creation

You can create a project with Teams Toolkit that comes with SSO support by default. Such as `SSO-enabled tab`. Refer [here](https://docs.microsoft.com/microsoftteams/platform/toolkit/create-new-project) for how to create a new Teams app with Teams Toolkit. An Microsoft Entra manifest file will be automatically created for you: `templates\appPackage\aad.template.json`. Teams Toolkit will create or update the Microsoft Entra application during local development or when you move your application to the cloud.

### 2. Add SSO to your Bot or Tab capability
If you created a Teams application without SSO built-in, Teams Toolkit can incrementally help you [add SSO](https://teamsfx-add-sso) to your project. As a result, An Microsoft Entra manifest file will be automatically created for you: `templates\appPackage\aad.template.json`. Teams Toolkit will create or update the Microsoft Entra application during next local debug session or when you move your application to the cloud.

### 3. Local development

During local development (known as `F5`), Teams Toolkit will:
* Check if there is an existing Microsoft Entra application by reading the `state.local.json` file. If yes, Teams Toolkit will re-use the existing Microsoft Entra application otherwise we will create a new one using the `aad.template.json` file.

* When creating a new Microsoft Entra application with the manifest file, some properties in the manifest file which require additional context (such as `replyUrls` property that requires a local debug endpoint) will be ignored first.

* After the local dev environment startup successfully, the Microsoft Entra application's `identifierUris`, `replyUrls`, and other properties which are not available during creation stage will be updated accordingly.

* Changes you made to your Microsoft Entra application will be loaded during next local debug session. Optionally you can follow this [instruction](How-to-deploy-Microsoft-Entra-application-changes-for-local-environment) if you want to manually apply Microsoft Entra application changes.

### 4. Provision cloud resources

When moving your application to the cloud, you would need to provision cloud resources and deploy your application. At these stages, like local development, Teams Toolkit will:

* Check if there is an existing Microsoft Entra application by reading `state.{env}.json` file. If yes, Teams Toolkit will re-use the existing Microsoft Entra application otherwise we will create a new one using the `aad.template.json` file.

* When creating a new Microsoft Entra application with the manifest file, some properties in the manifest file which require additional context (such as `replyUrls` property requires frontend or bot endpoint) will be ignored first.

* After other resources provision completes, the Microsoft Entra application's `identifierUris` and `replyUrls` will be updated accordingly to the correct endpoints.

### 5. Application deployment

* `Deploy to the cloud` command will deploy your application to the provisioned resources. This will not include deploying Microsoft Entra application changes you made.
* You can follow this [instruction](#How-to-deploy-Microsoft-Entra-application-changes-for-remote-environment) to deploy Microsoft Entra application changes for remote environment
* Teams Toolkit will update the Microsoft Entra application according to the Microsoft Entra manifest template file.
> Please note, when deploying Microsoft Entra application for remote environment, you need to trigger provision first.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>

## Known limitations

1. Not all the properties listed in [Microsoft Entra manifest schema](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest) are supported in Teams Toolkit extension, this tab show the properties that are not supported:

    | Not supported properties   | Reason                     |
    | -------------------------- | -------------------------- |
    | passwordCredentials        | Not allowed in manifest    |
    | createdDateTime            | Readonly and cannot change |
    | logoUrl                    | Readonly and cannot change |
    | publisherDomain            | Readonly and cannot change |
    | oauth2RequirePostResponse  | Doesn't exist in Graph API |
    | oauth2AllowUrlPathMatching | Doesn't exist in Graph API |
    | samlMetadataUrl            | Doesn't exist in Graph API |
    | orgRestrictions            | Doesn't exist in Graph API |
    | certification              | Doesn't exist in Graph API |

1. Currently `requiredResourceAccess` property can use user readable resource app name or permission name strings only for `Microsoft Graph` and `Office 365 SharePoint Online` APIs. For other APIs, you need to use uuid instead. You can follow these steps retrieve ids from Azure Portal:

    1. [Register a new Microsoft Entra application](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) on Azure Portal.
    1. Click `API permissions` from the Microsoft Entra application page.
    1. Click `Add a permission` to add the permission you want.
    1. Click `Manifest`, from the `requiredResourceAccess` property, you can find the ids of API and permissions.

<p align="right"><a href="#Microsoft-Entra-manifest-introduction">back to top</a></p>