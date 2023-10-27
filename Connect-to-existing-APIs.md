# Connect to existing APIs

> This feature is currently under active development. Report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

When building a Teams application often you will want to access existing APIs - these might be APIs developed by your organization or even 3rd party APIs. If you do not have language appropriate SDKs to access these APIs, Teams Toolkit helps you bootstrap sample code to access these APIs.

In this tutorial, you will:

* [Use Teams Toolkit to connect to an existing API](#How-to-use-this-feature)
* [Understand the changes to your project when you connect to an existing API](#Understanding-the-updates-to-your-project)
* [Invoke the API in the Teams Toolkit local environment](#Testing-your-API-connection-in-the-Teams-Toolkit-local-environment)
* [Add configuration to access the API when you deploy your application to Azure](#Deploy-your-application-to-Azure)
* [Use a custom authentication provider](#Custom-authentication-provider)
* [Obtain API permissions with Microsoft Entra protected API request](#Connecting-to-APIs-that-require-Microsoft-Entra-permissions)

## How to use this feature

When you use Teams Toolkit to connect to an existing API, Teams Toolkit will:

* Generate sample code under `./bot` or `./api` folder.
* Add a reference to the `@microsoft/teamsfx` package to `package.json`.
* Add application settings for your API in `.env.teamsfx.local`, which configures local debugging.

To add an API connection:

### In Visual Studio Code

1. Open a TeamsFx project
2. From the Teams Toolkit side bar select `Add features` or open the Visual Studio Code Command Palette and select `Teams: Add features`:

![image](https://user-images.githubusercontent.com/11220663/165349151-bf009c88-907a-4fd1-9d2d-cdb26e2a93cc.png)

2. Select `API Connection`:

![image](https://user-images.githubusercontent.com/11220663/165430594-3793de91-ac4b-4746-9810-e004aadea19b.png)

3. Input endpoint for your API. The endpoint should be a valid http(s) url. It will be added to your project's local application settings and it is the base url for API  requests.

![image](https://user-images.githubusercontent.com/11220663/165430748-9c61a7a0-e51e-4642-8735-04affdc2bf74.png)

4. Choose which component will access the API.

![image](https://user-images.githubusercontent.com/11220663/165430964-b5273e72-ee76-41f5-b103-3dcc440e17f7.png)

5. Input an alias for your API. This alias is used to generate the application settings name for your API, and it will be added to your project's local application settings.

![image](https://user-images.githubusercontent.com/11220663/165431010-28b22571-31a4-4fcc-9b90-e14626ef182c.png)

6. Select how you want to authenticate the API requests. We will generate appropriate sample code and add corresponding local application settings based on your selection.

![image](https://user-images.githubusercontent.com/11220663/165431048-66b522d3-c616-4b84-b7aa-41a14f92696a.png)

7. There will be some additional configuration required based on the authentication type selected.

### In TeamsFx CLI

The base command of this feature is `teamsfx add api-connection [authentication type]`. Here is the list of available authentication type and corresponding sample command. You can use `teamsfx add api-connection [authentication type] -h` to get help document.

| Authentication type | Sample command |
| --- | --- |
| Basic | teamsfx add api-connection basic --endpoint https://example.com --component bot --alias example --user-name exampleuser --interactive false |
| API Key | teamsfx add api-connection apikey --endpoint https://example.com --component bot --alias example --key-location header --key-name example-key-name --interactive false |
| Microsoft Entra | teamsfx add api-connection aad --endpoint https://example.com --component bot --alias example --app-type custom --tenant-id your_tenant_id --app-id your_app_id --interactive false |
| Certificate | teamsfx add api-connection cert --endpoint https://example.com --component bot --alias example --interactive false |
| Custom | teamsfx add api-connection custom --endpoint https://example.com --component bot --alias example --interactive false |

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>

## Understanding the updates to your project

Teams Toolkit will make following changes to `bot` or `api` folder based on your selections:

1. Generate `{your_api_alias}.js/ts`. The file initializes an API client for your API and exports the API client.
2. Add `@microsoft/teamsfx` package to `package.json`. The package provides support for common API authentication methods.
4. Add environment variables to `.env.teamsfx.local`. These are the configuration for the selected authentication type. The generated code will read values from these environment variables.

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>

## Testing your API connection in the Teams Toolkit local environment

After adding an API connection:

### 1. Run npm install

You need to run `npm install` under `bot` or `api` folder to install the added packages.

### 2. Add your API credentials to the local application settings

Teams Toolkit does not ask for credentials but it will leave placeholders in the local application settings file. Substitute these placeholders with the appropriate credentials to access the API. The local application settings file is the `.env.teamsfx.local` file in the `bot` or `api` folder.

### 3. Use the API client to make API requests
Import the API client from the source code that needs access to the API:

``` ts
import { yourApiClient } from '{relative path to the generated file}'
```

### 4. Make http(s) requests to target API (with Axios)
The generated API client is an Axios API client. Use the Axios client to make requests to the API.

[Axios](https://www.npmjs.com/package/axios) is a popular nodejs package for making http(s) requests. You can visit Axios [documentation](https://axios-http.com/docs/example) for more information.

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>

## Deploy your application to Azure

To deploy your application to Azure, you will need to add the authentication configuration to the application settings for the appropriate environment. For example, your API might have different credentials for `dev` and `prod`. You can configure Teams Toolkit appropriately based on your environment needs.

Teams Toolkit only configures your local environment. The bootstrapped sample code contains comments that tell you what app settings you need to configure. This [document](https://aka.ms/teamsfx-add-appsettings) contains more information on adding application settings.

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>

## Advanced scenarios

### Custom authentication provider

Besides the authentication provider included in `@microsoft/teamsfx` package, you can also implement your customized authentication provider that implements `AuthProvider` interface and use it in `createApiClient(..)` function:

``` ts
import { AuthProvider } from '@microsoft/teamsfx'

class CustomAuthProvider implements AuthProvider {
    constructor() {
        // You can add necessary parameters for your customized logic in constructor
    }

    AddAuthenticationInfo: (config: AxiosRequestConfig) => Promise<AxiosRequestConfig> = async (
        config
    ) => {
        /*
        * The config parameter contains all the request information and can be updated to include extra authentication info.
        * Refer https://axios-http.com/docs/req_config for detailed document for the config object.
        * 
        * Add your customized logic that returns updated config
        */
    };
}
```

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>

### Connecting to APIs that require Microsoft Entra permissions

Some services are authenticated by Microsoft Entra. To access these services, there are two common ways to configure the API permissions:
* Use Access Control Lists (ACLs)
* Use Microsoft Entra application permissions

Obtaining a token with the right resource scopes for your API depends on the implementation of the API. Here are the steps to access these APIs:

#### When using Microsoft Entra application permissions

1. Open `templates/appPackage/aad.template.json` and add the following to `requiredResourceAccess` property:
   ```
    {
        "resourceAppId": "The Microsoft Entra App Id for the service providing the API you are connecting to",
        "resourceAccess": [
            {
                "id": "Target API's application permission Id",
                "type": "Role"
            }
        ]
    }
   ```
2. Start local debug or provision an cloud environment for your project. This will create an Microsoft Entra Application Registration your Teams application.
3. Open `.fx/states/state.{env}.json` and note the value of `clientId` under `fx-resource-aad-app-for-teams` property. This is your Teams application client id.
4. Follow this [document](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/grant-admin-consent#grant-admin-consent-in-app-registrations) to gain admin consent for the required application permission. You will need your application client id.

#### When using Access Control Lists (ACsL)
1. Start local debug or provision an cloud environment for your project. This will create an Microsoft Entra Application Registration your Teams application.
2. Open `.fx/states/state.{env}.json` and note the value of `clientId` under `fx-resource-aad-app-for-teams` property.
3. Provide the client id to your API provider to configure ACLs on the API service.

<p align="right"><a href="#Connect-to-existing-APIs">back to top</a></p>
