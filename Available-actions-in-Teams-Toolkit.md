### The page describes the available actions that can be used in `teamsapp.yml` and `teamsapp.local.yml` in Teams Toolkit.

### Links to more information

For information on Teams Toolkit v5, visit [Teams Toolkit v5 guide](https://aka.ms/teamsfx-v5.0-guide).

For information on lifecycles, visit [Lifecycles](https://aka.ms/teamsfx-v5.0-guide#lifecycles).

For information on actions, visit [Actions](https://aka.ms/teamsfx-v5.0-guide#actions).

For information on environments, visit [Environments](https://aka.ms/teamsfx-v5.0-guide#environments).

# aadApp/create
This action will create a new Microsoft Entra app to authenticate users if the environment variable that stores clientId is empty.

## Overview

The `aadApp/create` action allows you to create a Microsoft Entra application with an optional client secret. This action generates identifiers such as `clientId`, `objectId`, `tenantId`, `authority`, and `authorityHost`, which are essential for managing the Microsoft Entra application. If required, a client secret can also be generated.

## Syntax:
```
  - uses: aadApp/create
    with:
      name: <your-application-name> # Required. when you run aadApp/update, the Microsoft Entra app name will be updated based on the definition in manifest. If you don't want to change the name, make sure the name in Microsoft Entra manifest is the same with the name defined here.
      generateClientSecret: true # Required. If the value is false, the action will not generate client secret for you
      signInAudience: "AzureADMyOrg" # Required. Specifies what Microsoft accounts are supported for the current application. Supported values are: `AzureADMyOrg`, `AzureADMultipleOrgs`, `AzureADandPersonalMicrosoftAccount`, `PersonalMicrosoftAccount`.
      clientSecretExpireDays: 180 # Optional. Only available in schema version `v1.5` and later. Specify the lifetime of a client secret. The default value is 180 days.
      clientSecretDescription: "some description" # Optional. Only available in schema version `v1.5` and later. Add a description for the client secret. The default value is `default`.
      serviceManagementReference: "some value" # Optional. Only available in schema version `v1.5` and later. References application or service contact information from a Service or Asset Management database.
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      clientId: <your-preferred-env-var-name> # Required. The client (application) ID of Microsoft Entra application. The action will refer the environment variable defined here to determine whether to create a new Microsoft Entra app.
      clientSecret: <your-preferred-env-var-name> # Required when `generateClientSecret` is `true`. The action will refer the environment variable defined here to determine whether to create a new client secret. It's recommended to add `SECRET_` prefix to the environment variable name so it will be stored to the .env.{envName}.user environment file.
      objectId: <your-preferred-env-var-name> # Required. The object ID of Microsoft Entra application
      tenantId: <your-preferred-env-var-name> # Optional. The tenant ID of Microsoft Entra tenant
      authority: <your-preferred-env-var-name> # Optional. The Microsoft Entra authority
      authorityHost: <your-preferred-env-var-name> # Optional. The host name of Microsoft Entra authority
```
## Input Specification

### YAML Example

**Creating an App Without a Client Secret:**
```yaml
name: Create My Entra App
uses: aadApp/create
with:
  name: MyApp
  generateClientSecret: false
  signInAudience: AzureADMyOrg
  serviceManagementReference: AppServiceRef
writeToEnvironmentFile:
  clientId: CLIENT_ID_ENV_VAR
  objectId: OBJECT_ID_ENV_VAR
  tenantId: TENANT_ID_ENV_VAR
  authority: AUTHORITY_ENV_VAR
  authorityHost: AUTHORITY_HOST_ENV_VAR
```

**Creating an App With a Client Secret:**
```yaml
name: Create My Entra App With Secret
uses: aadApp/create
with:
  name: MyAppWithSecret
  generateClientSecret: true
  signInAudience: AzureADMyOrg
  clientSecretExpireDays: 90
  clientSecretDescription: "My client secret"
writeToEnvironmentFile:
  clientId: CLIENT_ID_ENV_VAR
  objectId: OBJECT_ID_ENV_VAR
  clientSecret: CLIENT_SECRET_ENV_VAR
  tenantId: TENANT_ID_ENV_VAR
  authority: AUTHORITY_ENV_VAR
  authorityHost: AUTHORITY_HOST_ENV_VAR
```

### Input Details

- **name** (Required): Specifies the name of the Microsoft Entra application.
- **generateClientSecret** (Required): Indicates whether a client secret should be generated (`true` or `false`).
- **signInAudience** (Required): Specifies the supported Microsoft accounts. Possible values are:
  - `AzureADMyOrg`
  - `AzureADMultipleOrgs`
  - `AzureADandPersonalMicrosoftAccount`
  - `PersonalMicrosoftAccount`
- **serviceManagementReference** (Optional): Reference to an application or service contact information from a Service or Asset Management database.
- **clientSecretExpireDays** (Optional): Number of days the client secret is valid. Must be a positive integer.
- **clientSecretDescription** (Optional): Description of the client secret.

### Input Validation Rules

1. The `name` of the Microsoft Entra application must be a non-empty string and less than or equal to 120 characters.
2. The `generateClientSecret` must be a boolean value.
3. The `signInAudience` must be one of the allowed values.
4. The `clientSecretExpireDays`, if specified, must be a positive integer.

## Output Specification

The outputs are written to environment variables defined in the `writeToEnvironmentFile` object. The required environment variables include:

### Without Client Secret:

- `clientId`: The client (application) id of the created Microsoft Entra application.
- `objectId`: The object id of the created Microsoft Entra application.
- `tenantId`: The tenant id of the created Microsoft Entra application.
- `authority`: The authority of the created Microsoft Entra application.
- `authorityHost`: The authority host name of the created Microsoft Entra application.

### With Client Secret:

Includes the above variables and additionally:
- `clientSecret`: The generated client secret of the Microsoft Entra application.

## Potential Errors for Troubleshooting

### Error Classes and Reasons

1. **InvalidActionInputError**
   - **Reason**: Some of the required inputs are missing or have invalid values.
   - **Solution**: Ensure that all required inputs (`name`, `generateClientSecret`, `signInAudience`) are correctly specified. The `name` should not exceed 120 characters.

2. **AadAppNameTooLongError**
   - **Reason**: The provided `name` exceeds the limit of 120 characters.
   - **Solution**: Shorten the name of the Microsoft Entra application.

3. **OutputEnvironmentVariableUndefinedError**
   - **Reason**: The `writeToEnvironmentFile` object is not defined properly.
   - **Solution**: Define appropriate environment variables to capture the outputs.

4. **MissingEnvUserError**
   - **Reason**: `objectId` environment variable is not defined when attempting to generate a client secret.
   - **Solution**: Ensure the environment variable for `objectId` is provided.

5. **HttpClientError**
   - **Reason**: A client error occurred while making HTTP requests (status code 4xx).
   - **Solution**: Recheck the request parameters and ensure the API is accessible.

6. **HttpServerError**
   - **Reason**: A server error occurred (status code 5xx).
   - **Solution**: Retry the operation or check the service status.

7. **CredentialInvalidLifetimeError**
   - **Reason**: Provided client secret lifetime is invalid as per the application policy.
   - **Solution**: Adjust the `clientSecretExpireDays` according to the allowed lifetime.

8. **ClientSecretNotAllowedError**
   - **Reason**: Client secrets are not allowed as per the application policy.
   - **Solution**: Check the application policies and possibly alter the approach to authentications.

### General Troubleshooting Steps:

- Ensure all required parameters are provided and conform to the specified types and constraints.
- Check network connectivity and service availability if HTTP errors occur.
- Use the error messages and solution suggestions to debug and fix issues.

# aadApp/update
This action will update your Microsoft Entra app based on give Microsoft Entra app manifest. It will refer the `id` property in Microsoft Entra app manifest to determine which Microsoft Entra app to update.

## Overview

The `aadApp/update` action updates a Microsoft Entra application based on a provided application manifest. If the manifest uses `AAD_APP_ACCESS_AS_USER_PERMISSION_ID` and the corresponding environment variable is empty, this action will generate a random ID and output it.

## Syntax:
```
  - uses: aadApp/update
    with:
      manifestPath: ./aad.manifest.json # Required. Relative path to this file. Environment variables in manifest will be replaced before apply to Microsoft Entra app.
      outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json # Required. Relative path to teamsfx folder. This action will output the final Microsoft Entra manifest used to update Microsoft Entra app to this path.
```
## Input Validation Rules

The input arguments for this action are specified in the `with` object. All input parameters are mandatory:

- `manifestPath`: Path of the Microsoft Entra application manifest. Environment variables in the manifest will be replaced before applying the manifest to the Microsoft Entra application.
- `outputFilePath`: Path to generate the final Microsoft Entra application manifest used to update the application.

Example:
```yaml
with:
  manifestPath: "./path/to/manifest.json"
  outputFilePath: "./path/to/outputManifest.json"
```

### Validation Criteria
- `manifestPath` must be a non-empty string.
- `outputFilePath` must be a non-empty string.
- If any of these criteria are not met, an `InvalidActionInputError` will be raised.


## Output Specification

The output of the action execution will be stored in the environment file as specified by the `writeToEnvironmentFile` object:

- `AAD_APP_ACCESS_AS_USER_PERMISSION_ID`: The environment variable name to store the generated or existing permission ID.

## Errors and Troubleshooting

1. **InvalidActionInputError**
   - **Reason**: One or more required input parameters are missing or invalid.
   - **Possible Solution**: Ensure that `manifestPath` and `outputFilePath` are correctly provided and are non-empty strings.

2. **FileNotFoundError**
   - **Reason**: The specified manifest file does not exist.
   - **Possible Solution**: Verify that the file path provided in `manifestPath` points to a valid file.

3. **MissingFieldInManifestUserError**
   - **Reason**: The manifest is missing a required field (`id`).
   - **Possible Solution**: Check the manifest file to ensure all required fields are included.

4. **HttpClientError**
   - **Reason**: The request to the Microsoft Graph API resulted in a client error (4xx).
   - **Possible Solution**: Verify the manifest content and ensure that the application has the necessary permissions to update the Microsoft Entra application.

5. **HttpServerError**
   - **Reason**: The request to the Microsoft Graph API resulted in a server error (5xx).
   - **Possible Solution**: Retry the operation later. If the issue persists, contact Microsoft support.

6. **DeleteOrUpdatePermissionFailedError**
   - **Reason**: Error occurs when there is a failure in deleting or updating permissions.
   - **Possible Solution**: Check permissions and ensure the manifest content is correct. Retry the action after some time.

7. **HostNameNotOnVerifiedDomainError**
   - **Reason**: The specified hostname is not on a verified domain.
   - **Possible Solution**: Verify that the domain of the hostname is correctly configured and verified in Microsoft Entra.

### Other issues

#### Error message "Permission (scope or role) cannot be deleted or updated unless disabled first
This is a known issue that OAuth permission id for an existing permission in your Microsoft Entra manifest is different than the id in Microsoft Entra application. One possible reason is the value of `AAD_APP_ACCESS_AS_USER_PERMISSION_ID` environment variable in `.env.{env_name}` is out of sync.

To fix this error: find the id of `access_as_user` scope for your application in [Microsoft Entra app registration portal](https://ms.portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) and set it to `AAD_APP_ACCESS_AS_USER_PERMISSION_ID` environment variable in `.env.{env_name}`.

![image](https://user-images.githubusercontent.com/16605901/204182487-8eb46f6d-cee6-4d97-9cd4-68db59d4a572.png)

#### Expected property 'lang' is not present on resource of type 'Permissionscope'
When you use Microsoft Entra app manifest displayed in [Azure App Registration portal](https://ms.portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), you may meet this error or other similar errors. This is because the Microsoft Entra app manifest displayed in Azure Portal is not 100% compatible with [Microsoft Entra app manifest schema](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-app-manifest). This issue is being tracked and will be fixed in the future.

To fix this error: remove the extra properties mentioned in the error message and try to update your Microsoft Entra app again.

# teamsApp/create
This action will create a new Teams app for you if the environment variable that stores Teams app id is empty or the app with given id is not found from Teams Developer Portal.

## Syntax:
```
  - uses: teamsApp/create
    with:
      name: YOUR-APP-NAME-${{TEAMSFX_ENV}} # TEAMSFX_ENV is the environment variable defined in env/.env.<environment> file, used to differentiate Teams app in Teams Developer Portal
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      teamsAppId: TEAMS_APP_ID
```

## Output:
* teamsAppId: The id for Teams app
* TEAMS_APP_TENANT_ID: The tenant id for M365 account.

# teamsApp/update
Apply the Teams app manifest to an existing Teams app in Teams Developer Portal. Will use the app id in manifest.json file to determine which Teams app to update.

## Syntax:
```
  - uses: teamsApp/update # 
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
```

## Output:
* TEAMS_APP_TENANT_ID: The tenant id for M365 account.
* TEAMS_APP_UPDATE_TIME: The latest update time for syncing local manifest file to Teams Developer Portal.

# teamsApp/validateManifest
This action will render Teams app manifest template with environment variables, validate Teams app manifest file using its schema.

## Syntax:
```
  - uses: teamsApp/validate
    with:
      manifestPath: ./appPackage/manifest.json # Required. Path to Teams app manifest file
```

## Output:
* NONE

# teamsApp/validateAppPackage
This action will validate Teams app package using validation rules.

## Syntax:
```
  - uses: teamsApp/validateAppPackage
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
```

## Output:
* NONE

# teamsApp/validateWithTestCases
This action will send out async validation requests to the Developer Portal to validate the app package.

## Syntax:
```
  - uses: teamsApp/validateWithTestCases
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
      showMessage: true # Optional. Show message or not.
      showProgressBar: true # Optional. Show progress bar or not.
```

## Output:
* NONE

# teamsApp/zipAppPackage
This action will render Teams app manifest template with environment variables, and zip manifest file with two icons.

## Syntax:
```
  - uses: teamsApp/zipAppPackage
    with:
      manifestPath: ./appPackage/manifest.json # Required. Relative path to this file. This is the path for Teams app manifest file.
      outputZipPath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
      outputJsonPath: ./appPackage/build/manifest.${{TEAMSFX_ENV}}.json # Required. Relative path to this file. This is the path for built manifest json file.
```

## Output:
* NONE

# teamsApp/publishAppPackage
This action will publish built Teams app zip file to tenant app catalog.

## Syntax:
```
  - uses: teamsApp/publishAppPackage
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      publishedAppId: TEAMS_APP_PUBLISHED_APP_ID
```

## Output:
* publishedAppId: The Teams app id in tenant app catalog. It is different from the app id in Teams developer Portal.

# azureStorage/enableStaticWebsite
This action will enable static website setting in Azure Stroage.

## Syntax:
```
  - uses: azureStorage/enableStaticWebsite
    with:
      storageResourceId: ${{TAB_AZURE_STORAGE_RESOURCE_ID}}
      indexPage: index.html
      errorPage: error.html
```

## Output:
NA

# azureStaticWebApps/getDeploymentToken
This action will get deployment token from Azure Static Web Apps.

## Version Info
Since the Yaml schema v1.4

## Syntax:
```
  - uses: azureStaticWebApps/getDeploymentToken
    with:
      resourceId: ${{AZURE_STATIC_WEB_APPS_RESOURCE_ID}}
    writeToEnvironmentFile:
      deploymentToken: SECRET_TAB_SWA_DEPLOYMENT_TOKEN
```
## Output:
* The deployment token of the Azure Static Web Apps.

## Troubleshooting:
* The deployment token will persist until it is reset. If you have reset the token, you must run this action again or update the deployment token in your .env file.

# cli/runNpxCommand
This action will execute `npx` commands under specified directory with parameters. It can be used to run `gulp` commands to bundle and package sppkg.

## Syntax:
```
  - uses: cli/runNpxCommand
    with:
      workingDirectory: ./src
      args: gulp bundle --ship --no-color
  - uses: cli/runNpxCommand
    with:
      workingDirectory: ./src
      args: gulp package-solution --ship --no-color
```

## Output:
* A client-side solution package that is located in `{workingDirectory}`/sharepoint/solution/*.sppkg

## Troubleshooting:
There's a known issue that `gulp bundle --ship --no-color` command will fail with exit code 1 if there's lint errors in your SPFx project. You'll need to fix them to continue although they're displayed as warning in log details.


# cli/runNpmCommand
This action will execute `npm` commands under specified directory with parameters. The parameter `workingDirectory` can be removed if you want to run this command in the project root.

## Syntax:
```
  - uses: cli/runNpmCommand
    with:
      args: run build
  - uses: cli/runNpmCommand
    with:
      workingDirectory: ./src
      args: install
```

## Output:
NA

## Troubleshooting:
### Error message "Failed to run command"
Please check if the command exists in your system path and try to run this command manually in your working directory.


# cli/runDotnetCommand
This action will execute `dotnet` commands under specified directory with parameters. The parameter `workingDirectory` can be removed if you want to run this command in the project root.

## Syntax:
```
  - uses: cli/runDotnetCommand
    with:
      args: publish
  - uses: cli/runDotnetCommand
    with:
      workingDirectory: ./src
      execPath: /YOU_DOTNET_INSTALL_PATH
      args: publish --configuration Release --runtime win-x86 --self-contained
```

## Output:
NA

## Troubleshooting:
### Error message "Failed to run command"
Please check if the command exists in your system path and try to run this command manually in your working directory.

# azureAppService/zipDeploy
This action will upload and deploy the project to Azure App Service using [the zip deploy feature](https://aka.ms/zip-deploy-to-app-services). 
The parameter `workingDirectory` refers to the root folder for deploy action operations. It can be removed if you want to run deploy command in the project root.

The `artifactFolder` parameter represents the folder where you want to upload the artifact. If your input value is a relative path, it is relative to the `workingDirectory`.

The `ignoreFile` parameter specifies the file path of the ignore file used during upload. This file can be utilized to exclude certain files or folders from the `artifactFolder`. Its syntax is similar to the Git's ignore.

The `resourceId` parameter indicates the resource ID of an Azure App Service. It is generated automatically after running the provision command. If you already have an Azure App Service, you can find its resource ID in the Azure portal (see this [link](https://azurelessons.com/how-to-find-resource-id-in-azure-portal/) for more information).

You can set the `dryRun` parameter to true if you only want to test the preparation of the upload and do not intend to deploy it. This will help you verify that the packaging zip file is correct. The default value for this parameter is false.

The `outputZipFile` parameter indicates the path of the zip file for the packaged artifact folder. It is relative to the `workingDirectory`, and its default value is `.deployment/deployment.zip`. This file will be reconstructed during deployment, reflecting all folders and files in your `artifactFolder`, and removing any non-existent files or folders.

## Syntax:
```
  - uses: azureAppService/zipDeploy
    with:
      workingDirectory: ./src
      artifactFolder: .
      ignoreFile: ./.webappignore
      resourceId: ${{BOT_AZURE_APP_SERVICE_RESOURCE_ID}}
      dryRun: false
      outputZipFile: ./.deployment/deployment.zip
```

## Output:
NA

## Troubleshooting:
### Error message: No file is found in distribution folder
Please make sure the artifactFolder is not empty.

### Error message: Failed to list publishing credentials.
Please retry first, if it does not work, please check your Azure account and make sure this account can use [this api](https://learn.microsoft.com/en-us/rest/api/appservice/web-apps/list-publishing-credentials#code-try-0). You can test it in the right side of the page.

### Error message: Remote service error, upload failed.
Please wait for a while before retrying.

### Error message: Failed to deploy zip file.
Please check the log output and try to upload the files located in your .deployment folder which is in your artifact folder according to the guidelines in [this link](https://learn.microsoft.com/en-us/azure/app-service/deploy-zip?tabs=kudu-ui).

### Error message: Failed to check deployment status.
This error can be ignored if the deployment is already successful. You can check the deploy status by visiting `Deployment - Deployment center - Logs` in the Azure portal.

### Error message: Error: Request failed with status code 401
This error may have two possible causes:

1. Your Azure login information is incorrect, and you are attempting to deploy to an App Service that you do not have permission for. Please double-check that you have enough permissions for this resource.
2. You are using an old version of the Teams Toolkit while your Web App has Basic Auth turned off. Log into the Azure Portal, find your Web App, then go to Settings - Configuration - General Settings - Basic Authentication and set it to ON before trying to redeploy again.

# azureFunctions/zipDeploy
This action will upload and deploy the project to Azure Functions using [the zip deploy feature](https://aka.ms/zip-deploy-to-azure-functions). 
The parameter `workingDirectory` refers to the root folder for deploy action operations. It can be removed if you want to run deploy command in the project root.

The `artifactFolder` parameter represents the folder where you want to upload the artifact. If your input value is a relative path, it is relative to the `workingDirectory`.

The `ignoreFile` parameter specifies the file path of the ignore file used during upload. This file can be utilized to exclude certain files or folders from the `artifactFolder`. Its syntax is similar to the Git's ignore.

The `resourceId` parameter indicates the resource ID of an Azure Function. It is generated automatically after running the provision command. If you already have an Azure Function, you can find its resource ID in the Azure portal (see this [link](https://azurelessons.com/how-to-find-resource-id-in-azure-portal/) for more information).

You can set the `dryRun` parameter to true if you only want to test the preparation of the upload and do not intend to deploy it. This will help you verify that the packaging zip file is correct. The default value for this parameter is false.

The `outputZipFile` parameter indicates the path of the zip file for the packaged artifact folder. It is relative to the `workingDirectory`, and its default value is `.deployment/deployment.zip`. This file will be reconstructed during deployment, reflecting all folders and files in your `artifactFolder`, and removing any non-existent files or folders.

## Syntax:
```
  - uses: azureFunctions/zipDeploy
    with:
      workingDirectory: ./src
      artifactFolder: .
      ignoreFile: ./.webappignore
      resourceId: ${{BOT_AZURE_APP_SERVICE_RESOURCE_ID}}
      dryRun: false
      outputZipFile: ./.deployment/deployment.zip
```

## Output:
NA

## Troubleshooting:
### Error message: No file is found in distribution folder
Please make sure the `artifactFolder` path is not empty.

### Error message: Failed to list publishing credentials.
Please retry first, if it does not work, please check your Azure account and make sure this account can use [this api](https://learn.microsoft.com/en-us/rest/api/appservice/web-apps/list-publishing-credentials#code-try-0). You can test it in the right side of the page.

### Error message: Remote service error, upload failed.
Please wait for a while before retrying.

### Error message: Failed to deploy zip file.
Please check the log output and try to upload the files located in your .deployment folder which is in your artifact folder according to the guidelines in [this link](https://learn.microsoft.com/en-us/azure/app-service/deploy-zip?tabs=kudu-ui).

### Error message: Failed to check deployment status.
This error can be ignored if the deployment is already successful. You can check the deploy status by visiting `Deployment - Deployment center - Logs` in the Azure portal.

### Error message: remote server error with status code: 502
If the package you attempted to deploy is larger than 500MB and your Azure Functions SKU is set to Free, you may receive this notification. Please [refer this document for more information.](https://github.com/projectkudu/kudu/wiki/Understanding-the-Azure-App-Service-file-system#temporary-files).


# azureStorage/deploy
This action will upload and deploy the project to Azure Storage. The parameter `workingDirectory` can be removed if you want to run this command in the project root.

## Syntax:
```
  - uses: azureStorage/deploy
    with:
      workingDirectory: ./src
      artifactFolder: . # Deploy base folder
      ignoreFile: ./.webappignore # Can be changed to any ignore file location, leave blank will ignore nothing
      resourceId: ${{BOT_AZURE_APP_SERVICE_RESOURCE_ID}} # The resource id of the cloud resource to be deployed to
```

## Output:
NA

## Troubleshooting:
### Error message: Failed to clear Azure Storage Account.
Please retry later or you can clear all files in your $web container and retry the deployment action.

### Error message: Failed to upload local path xxxx to Azure Storage Account.
Please retry this action later.


# spfx/deploy
This action will upload and deploy generated sppkg to SharePoint app catalog. You can create tenant app catalog manually or by setting `createAppCatalogIfNotExist` to true if you don't have one in current M365 tenant.

## Syntax:
```
  - uses: spfx/deploy
    with:
      createAppCatalogIfNotExist: false # Required. If the value is true, this action will create tenant app catalog first if not exist, default value is `false`.
      packageSolutionPath: ./src/config/package-solution.json # Required. Path to package-solution.json in SPFx project. This action will honor the configuration to get target sppkg.
```

## Output:
NA

## Troubleshooting:
### Error message: Failed to create tenant app catalog.
Please retry later or create SharePoint app catalog manually.

# teamsApp/copyAppPackageToSPFx
This action will copy the Teams App zipped package to `teams` folder in SPFx directory to keep it updated. This is to ensure user will have aligned experience whether to publish Teams App from Teams Toolkit or manually sync to Teams in SharePoint app catalog.

## Syntax:
```
  - uses: teamsApp/copyAppPackageToSPFx
    with:
      appPackagePath: ${{TEAMS_APP_PACKAGE_PATH}}
      spfxFolder: ./src # Path to SPFx solution.
```

## Output:
NA

# teamsApp/extendToM365
This action will upload your app as M365 title, so it can be viewed on Outlook and Office.

## Syntax
```yml
  - uses: teamsApp/extendToM365 # Extend your Teams app to Outlook and the Microsoft 365 app
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to the built app package.
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      titleId: M365_TITLE_ID # Required. The ID of M365 title.
      appId: M365_APP_ID # Required. The app ID of M365 title.
```

# file/createOrUpdateEnvironmentFile
This action will create or update variables to environment file.

## Syntax:
```yml
  - uses: file/createOrUpdateEnvironmentFile
    with: 
      target: /path/to/your/env/file # Required. This action will generate envs to the specified path.
      envs: 
        <your-env-key-1>: <your-env-value-1>
        <your-env-key-2>: <your-env-value-2>
```

## Output:
NA

# file/createOrUpdateJsonFile
This action will create or update appsettings to JSON file.

## Syntax:
```yml
  - uses: file/createOrUpdateJsonFile
    with:
      target: ./appsettings.Development.json # Required. The relative path of settings file
      appsettings: # Required. The appsettings to be generated
        BOT_ID: ${{BOT_ID}}
        BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
```

## Output:
NA

# file/updateJson

> This action is deprecated. Please use [file/createOrUpdateJsonFile](#filecreateorupdatejsonfile) instead.

This action will override or add application settings to target file, in JSON format (e.g., appsettings.Development.json)

## Syntax:
```yml
  - uses: file/updateJson
    with:
      target: ./appsettings.Development.json # Required. The relative path of settings file
      appsettings: # Required. The appsettings to be generated
        BOT_ID: ${{BOT_ID}}
        BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
```

## Output:
NA

# botFramework/create
This action will create or update the bot registration on [dev.botframework.com](https://dev.botframework.com/bots). If the bot registraion specified by `botId` does not exist, this action will create a new one; otherwise, this action will update it.

> Note:
>
> There's a known issue for error "*'MsaAppTenantId' property cannot be changed*" when using an existing single-tenant bot. Currently `botFramework/create` only supports multi-tenant bot.
>
> To use single-tenant bot for local development, please:
> - Remove or comment out `botFramework/create` action from *teamsapp.local.yml* file
> - Start debug / F5
> - Get `BOT_ENDPOINT` from *env/.env.local* and manually update the bot's message endpoint to "*{{**BOT_ENDPOINT**}}/api/messages*"

## Syntax:
```yml
  - uses: botFramework/create
    with: 
      botId: <your-microsoft-app-id> # Required. Microsoft App Id for the bot registration.
      name: <your-bot-name> # Required. The name of the bot registration.
      messagingEndpoint: <your-messaging-endpoint> # Required. The messaging endpoint of the bot registration.
      description: <your-description> # Optional. The description of the bot registration.
      iconUrl: <your-icon-url> # Optional. The icon url of the bot registration.
      channels: # Optional. The channel configurations of the bot registration.
        - name: msteams # Required. The name of Microsoft Teams channel.
          callingWebhook: <your-calling-webhook> # Optional. The calling webhook of Microsoft Teams channel.
        - name: m365extensions # Required. The name of Microsoft 365 Extensions channel.
```

## Output:
NA

# file/updateEnv

> This action is deprecated. To generate to Teams Toolkit env file, please use [script](#script) to output. To generate to your own file, please use [file/createOrUpdateEnvironmentFile](#filecreateorupdateenvironmentfile) instead.

This action will generate environment variables to `.env` file.

## Syntax:
```yml
  - uses: file/updateEnv
    with: 
      target: /path/to/your/.env/file # Optional. If specified, this action will generate envs to the specified path. If not specified, this action will regard envs as outputs which will be persisted in current environment's .env file.
      envs: 
        <your-env-key-1>: <your-env-value-1>
        <your-env-key-2>: <your-env-value-2>
```

## Output:
If `target` is specified, NA. If `target` is not specified, `envs`.

# devTool/install
This action will install the development tool(s) required to debug a Teams app.

## Syntax:
```yml
  - uses: devTool/install # Install development tool(s)
    with:
      devCert: # Optional. The SSL certificate for Teams Tab app. This action will generate a SSL certificate and install it to the system certificate management center.
        trust: true # Required. Whether to trust the SSL certificate.
      func: # Optional. Azure Functions Core Tools.
        version: ~4.0.4670 # Required. The version number of Azure Functions Core Tools that follow the Semantic Versioning scheme.
        symlinkDir: ./devTools/func # Optional. The path of the symlink target for the folder containing Azure Functions Core Tools binaries.
      dotnet: true # Optional. .NET SDK.
    writeToEnvironmentFile: # Write the information of installed development tool(s) into environment file for the specified environment variable(s).
      sslCertFile: SSL_CRT_FILE # Optional. The path of the certificate file of the SSL certificate. This parameter takes effect only when `devCert` is specified.
      sslKeyFile: SSL_KEY_FILE # Optional. The path of the key file of the SSL certificate. This parameter takes effect only when `devCert` is specified.
      funcPath: FUNC_PATH # Optional. The path of the Azure Functions Core Tools binary. This parameter takes effect only when `func` is `true`.
      dotnetPath: DOTNET_PATH # Optional. The path of the .NET binary. This parameter takes effect only when `dotnet` is `true`.
```
## Troubleshooting:
### Manually install development tools
In case the Teams Toolkit fails to install prerequisites for you, you can manually install them by following the guidelines below.
#### How to install .NET SDK

Go to [the official website](https://dotnet.microsoft.com/download) to download and install the supported version:

| Platform | .NET versions |
| --- | --- |
| Windows, macOS (x64), Linux | **.NET Core 3.1 SDK (recommended)**, .NET 5.0 SDK, .NET 6.0 SDK  |
| macOS (arm64) | .NET 6.0 SDK |

> Note: Please restart all your Visual Studio Code instances after the installation is finished.

#### How to install Azure Functions Core Tools

Go to [the official website](https://github.com/Azure/azure-functions-core-tools) to install the `Azure Functions Core Tools v4`.

> Note: Please restart all your Visual Studio Code instances after the installation is finished.

#### How to install Teams App Test Tool

If `testTool/install` action fails to automatically install [Teams App Test Tool](https://www.npmjs.com/package/@microsoft/teams-app-test-tool), you can disable the action and install it manually.

Please run the following command to install the Test Tool globally.

```
npm install -g @microsoft/teams-app-test-tool
```

You can run `teamsapptester --version` to verify the installation.

> Note: Please restart all your Visual Studio Code instances after the installation is finished.

#### How to install Teams App Test Tool for Visual Studio

If Teams Toolkit for Visual Studio failed to download and install Teams App Test Tool automatically, you can install it by yourself and set its path in Tools > Options > Teams Toolkit:

1. Download the latest binary version of Teams App Test Tool from [here](https://github.com/OfficeDev/TeamsFx/releases?q=teams-app-test-tool&expanded=true).
1. Unzip the downloaded package and you can see an exe binary file named `teamsapptester.exe`.
1. In Visual Studio's menu bar, click Tools > Options > Teams Toolkit, then set Test Tool's installation path with the absolute path of the `teamsapptester.exe`:  
    
    <img src="https://github.com/OfficeDev/TeamsFx/assets/15644078/7985ec97-99f7-453c-b319-d5ab1154e2b2" height="320px"/>

> Note: Once the installation path of Test Tool is set, Teams Toolkit for Visual Studio won't try to download and install the latest version of Test Tool anymore.

# arm/deploy
This action will deploy given ARM templates parallelly

## Syntax
``` yaml
  - uses: arm/deploy
    with:
      subscriptionId: ${{AZURE_SUBSCRIPTION_ID}} # Required. You can use built-in environment variable `AZURE_SUBSCRIPTION_ID` here. TeamsFx will ask you select one subscription if its value is empty. You're free to reference other environment variable here, but TeamsFx will not ask you to select subscription if it's empty in this case.
      resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}} # Required. You can use built-in environment variable `AZURE_RESOURCE_GROUP_NAME` here. TeamsFx will ask you to select or create one resource group if its value is empty. You're free to reference other environment variable here, but TeamsFx will not ask you to select or create resource group if it's empty in this case.
      templates:
      - path: ./infra/azure.bicep # Required. Relative path to teamsfx folder.
        parameters: ./infra/azure.parameters.json # Required. Relative path to teamsfx folder. TeamsFx will replace the environment variable reference with real value before deploy ARM template.
        deploymentName: your-deployment-name # Required. Name of the ARM template deployment.
      bicepCliVersion: v0.9.1 # Optional. Teams Toolkit will download this bicep CLI version from github for you, will use bicep CLI in PATH if you remove this config.
```

## Output
This action will covert ARM deployment output to environment variables, with following naming conversion rule for output names:

1. Alphabet characters will be converted to upper case
2. Non alphanumeric character will be converted to `_`
3. If output is a hierarchy object, elements in the hierarchy is separated by a double underscore `__`

Taking following bicep output as example
```
output endpoint string = 'example'
output all_resource_ids object = {
  azureWebApp: {
    apiResourceId: 'web app id 1'
    frontendResourceId: 'web app id 2'
  }
  azureStorageId: 'storage id'
}
```
They will be outputted as following environment variables
``` env
ENDPOINT=example
ALL_RESOURCE_IDS__AZUREWEBAPP__APIRESOURCEID=web app id 1
ALL_RESOURCE_IDS__AZUREWEBAPP__FRONTENDRESOURCEID=web app id 2
ALL_RESOURCE_IDS__AZURESTORAGEID=storage id
```

## Troubleshooting

If you see errors from this action. You can follow below steps to find detailed error message:
1. Select the `Teams toolkit` channel of the output .
1. Find the execution summary at the bottom of the output channel.
1. Get the error code and error message from ARM.

Here're some common errors from ARM you might meet with Teams Toolkit's project templates.

### Website with given name xxx already exists.
### The storage account named xxx already exists under the subscription.
### The name 'xxx' already exists. Choose a different name.
Mitigation:

This above errors indicate the name for one or multiple Azure resources that going to be created already exists. The default name for all Azure resources is calculated based on the `resourceBaseName` parameter in `azure.parameters.{envName}.json`. Please update the value of `resourceBaseName` to fix this error.

### The subscription registration is in 'Unregistered' state. The subscription must be registered to use namespace 'xxx'.
This error indicates your Azure account does not have required permission to register the namespace. There are two ways to mitigate this issue:

Mitigation option 1:

Switch to an Azure account that has subscription level Contributor role.

Mitigation option 2:

Ask your subscription administrator to register the namespace mentioned in the error message by following this [link](https://aka.ms/rps-not-found).

# botAadApp/create
This action will create a new or reuses an existing Microsoft Entra application for bot.

## Syntax:
```yaml
  - uses: botAadApp/create # Creates a new or reuses an existing Microsoft Entra application for bot.
    with:
      name: {{appName}}-${{TEAMSFX_ENV}} # The Microsoft Entra application's display name
    writeToEnvironmentFile:
      botId: BOT_ID # The Microsoft Entra application's client id created for bot.
      botPassword: SECRET_BOT_PASSWORD # The Microsoft Entra application's client secret created for bot. 
```

# script
This action will execute a user defined script.

## Syntax:
```
  - uses: script
    with:
     run: $my_key="abc"; echo "::set-teamsfx-env mykey=${my_key}" # command to run or path to the script. Succeeds if exit code is 0. '::set-teamsfx-env key=value' is a special command to generate output variables into .env file, in this case, "mykey=abc" will be added the output in the corresponding .env file.
     workingDirectory: ./scripts # current working directory. Defaults to the directory of this file.
     shell: shell comand.
     timeout: 1000 # timeout in ms
     redirectTo: paht/to/file # redirect stdout and stderr to a file
```

## Output:
All stdout start with "::set-teamsfx-env key=value" will be interpreted into outputs in .env file.

## Default shell command
If `shell` is not specified, use default shell. The rule is applied in the following order:
1. Use the value of the 'SHELL' environment variable if it is set. 
2. If current OS is macOS, then use '/bin/zsh' if it exists, otherwise use '/bin/bash'; 
3. If current OS is Windows, then use the value of the 'ComSpec' environment variable if it exists, otherwise use 'cmd.exe'; 
4. If current OS is Linux or other OS systems, use '/bin/sh' if it exists. 

# apiKey/register
This action will register an API key in Developer Portal for authentication of API based message extension.
 
## Syntax:
```
  - uses: apiKey/register
    with:
      name: <your-api-key-name> # Required. Make sure the API key name in API specification is the same with the name defined here.
      appId: <your-teams-app-id> # Required. The id for Teams app you want to allow access to the API key.
      primaryClientSecret: <your-api-key-secret> # Optional. The client secret of your API key. Length of client secret >= 10 and <= 128
      secondaryClientSecret: <your-api-key-secret> # Optional. The client secret of your API key. Length of client secret >= 10 and <= 128
      apiSpecPath: <your-api-spec-path> # Required. Relative path to this file.
      applicableToApps: <applicableToApps-setting-of-your-api-key> # Optional. Choose which apps can use this Api Key. Values: SpecificApp, AnyApp.
      targetAudience: <targetAudience-setting-of-your-api-key> # Optional. Choose which tenant can use this API Key. Values: HomeTenant, AnyTenant
    writeToEnvironmentFile:
      registrationId: <your-preferred-env-var-name> # Required. The registration id of the API key.
```

# apiKey/update
This action will update an API key in Developer Portal for authentication of API based message extension.
 
## Syntax:
```
  - uses: apiKey/update
    with:
      name: <your-api-key-name> # Required. Make sure the API key name in API specification is the same with the name defined here.
      appId: <your-teams-app-id> # Required. The id for Teams app you want to allow access to the API key.
      apiSpecPath: <your-api-spec-path> # Required. Relative path to this file.
      registrationId: <your-registraion-id> # Required. The registration id of the Api key.
      applicableToApps: <applicableToApps-setting-of-your-api-key> # Optional. Choose which apps can use this Api Key. Values: SpecificApp, AnyApp.
      targetAudience: <targetAudience-setting-of-your-api-key> # Optional. Choose which tenant can use this API Key. Values: HomeTenant, AnyTenant
```

# General Errors
## ActionNotFoundError
This error means there's an unknown action in the yaml file. Please check whether the action type in 'uses' fields are supported. 
## YamlParsingError
This error means the syntax of the yaml file is invalid. Please check your syntax again.
## InvalidLifecycleError
This error means the format of the lifecycle is invalid. A lifecycle needs to be a yaml map. A typical example of this error is shown as follows:
```yaml
provision:
```
In this case, the value of lifecycle 'provision' will be interpreted as an empty string, which is invalid. If you want to remove all actions in provision, please remove it entirely, including the 'provision:' line.

Another possibile reason for this error is that the version of teamsapp.yml/teamspp.local.yml is not supported by your current Teams Toolkit. Please upgrade Teams Toolkit to latest version and try again.

## InvalidEnvFolderPath
This error means the 'environmentFolderPath' field is invalid. Please make sure it's a valid path.
## InvalidEnvFieldError
This error means the 'env' field of an action is invalid. 'env' field is used to define environment variables for a certain action. So, it's expected to contain key-value pairs, whose value is of type string.
Here is a valid example
```yaml
  - uses: botAadApp/create # Creates a new Microsoft Entra app for Bot Registration.
    env:
        BOT_ID: SOME_FAKE_ID
    with:
      name: bot # The display name of bot.
```
Below is an invalid example.
```yaml
  - uses: botAadApp/create # Creates a new Microsoft Entra app for Bot Registration.
    env:
        BOT_ID: 123 # 123 is a number, not a string.
    with:
      name: bot # The display name of bot.
```