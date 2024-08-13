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

## Overview

The `azureStorage/enableStaticWebsite` action configures an Azure Storage account to host a static website. Upon successful execution, the storage account will be set up to serve web content directly from Azure, making it suitable for hosting static web pages, single-page applications, and more.

## Syntax:
```
  - uses: azureStorage/enableStaticWebsite
    with:
      storageResourceId: ${{TAB_AZURE_STORAGE_RESOURCE_ID}}
      indexPage: index.html
      errorPage: error.html
```

## Input Parameters

The input parameters for this action are defined within the `with` object. Below are the parameters along with their validation rules:

- `storageResourceId` (string, required): The resource ID of the Azure Storage account.
- `indexPage` (string, optional): The path to the index page of the static website (default: `index.html`).
- `errorPage` (string, optional): The path to the error page of the static website (default: `index.html`).

### Example Input in YAML

```yaml
- uses: azureStorage/enableStaticWebsite
  with:
    storageResourceId: "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Storage/storageAccounts/{storage-account-name}"
    indexPage: "index.html"
    errorPage: "404.html"
```
## Potential Errors and Troubleshooting

Here are some common errors you might encounter while executing this action, along with their reasons and possible solutions:

### Error Classes

1. **InvalidInputError**
   - **Reason**: The `storageResourceId` parameter is missing or malformed.
   - **Solution**: Ensure the `storageResourceId` is provided and correctly formatted.

2. **AuthenticationError**
   - **Reason**: Authentication to Azure failed.
   - **Solution**: Verify that your Azure credentials are valid and have sufficient permissions to modify the specified storage account.

3. **ConfigurationError**
   - **Reason**: The static website configuration could not be applied, possibly due to an invalid index or error page path.
   - **Solution**: Double-check the paths for `indexPage` and `errorPage` to ensure they exist and are properly configured.

4. **NetworkError**
   - **Reason**: Network issues while connecting to Azure services.
   - **Solution**: Check your network connection and retry the operation.

### Example Error Message

If an error occurs, the output might include a message similar to this:

```text
Error: Invalid storageResourceId parameter. Please provide a valid Azure Storage resource ID.
```

# azureStaticWebApps/getDeploymentToken

## Overview

The `azureStaticWebApps/getDeploymentToken` action is designed to retrieve the deployment token for an Azure Static Web App. This token is critical for deploying code to the Static Web App from a CI/CD pipeline. The action fetches the deployment token using the provided resource ID of the Azure Static Web App and writes it to an environment file for further usage.

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

## Input Specification

The input arguments for this action are encapsulated within the `with` object. Below are the input validation rules and example inputs.

### Required Inputs

- **resourceId**: The resource ID of the Azure Static Web App.
  - **Type**: `string`
  - **Description**: This is a unique identifier for the Azure Static Web App.

### Example YAML Input

```yaml
uses: azureStaticWebApps/getDeploymentToken
with:
  resourceId: "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Web/staticSites/{site-name}"
writeToEnvironmentFile:
  deploymentToken: SWA_DEPLOYMENT_TOKEN
```


## Output:
- **deploymentToken**: The deployment token of the Azure Static Web App.

## Troubleshooting:
* The deployment token will persist until it is reset. If you have reset the token, you must run this action again or update the deployment token in your .env file.

1. **PrerequisiteError**
   - **Reason**: Invalid or improperly formatted `resourceId`.
   - **Solution**: Ensure that the `resourceId` follows the correct format: `/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Web/staticSites/{site-name}`.

2. **OutputEnvironmentVariableUndefinedError**
   - **Reason**: The `deploymentToken` key is not defined in `outputEnvVarNames`.
   - **Solution**: Verify that `deploymentToken` is specified correctly in the `writeToEnvironmentFile` object.

3. **BaseComponentInnerError**
   - **Reason**: General internal error.
   - **Solution**: Check the logs for detailed error messages and possible fixes. If the problem persists, consult the documentation or support.

4. **ExternalApiCallError**
   - **Reason**: Failures related to Azure credential retrieval.
   - **Solution**: Ensure you are logged into Azure and that the credentials are correct. Retry the operation later if remote errors persist.

5. **UserError or SystemError**
   - **Reason**: General user or system errors.
   - **Solution**: Read the detailed error messages and follow suggestions for resolving issues.
# cli/runNpxCommand

## Overview
The `cli/runNpxCommand` action is designed to execute an `npx` command with specified arguments and within a given working directory. The command execution is monitored and telemetry data are collected for analysis. This action is particularly useful for running scripts or commands available through `npx` in a controlled CI/CD environment.

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
## Input Validation Rules
The input for the `cli/runNpxCommand` action is specified through the `with` object. The following properties are needed:

- **args** (string; required): The arguments that will be passed to the `npx` command.
- **workingDirectory** (string; optional): The working directory where the command should be executed. Defaults to './' if not specified.

### Example Input
Below is an example of the `cli/runNpxCommand` action input in YAML format:

```yaml
uses: cli/runNpxCommand
with:
  args: "create-react-app my-app"
  workingDirectory: "./my-working-directory"
```

## Output:
* A client-side solution package that is located in `{workingDirectory}`/sharepoint/solution/*.sppkg

## Potential Errors and Troubleshooting
This section enumerates common errors users might encounter, along with their reasons and possible solutions.

### PrerequisiteError
- **Error Class**: `PrerequisiteError`
- **Reason**: Missing required argument such as `args`.
- **Solution**: Ensure that the `args` attribute is provided in the `with` section of the action input. Example:

  ```yaml
  with:
    args: "your-command-here"
  ```

### ScriptExecutionError
- **Error Class**: `ScriptExecutionError`
- **Reason**: The command failed during execution. This could occur due to syntax errors, missing dependencies, or issues within the script being run.
- **Solution**: Check the error message and the script for potential issues. Ensure all dependencies are correctly installed. Error messages can be found in the `npx_error` output.

### ScriptTimeoutError
- **Error Class**: `ScriptTimeoutError`
- **Reason**: The command took longer than the allowed time to execute and was terminated.
- **Solution**: Optimize the script to run within the time limit or increase the timeout duration if possible.

### EnvironmentError
- **Error Class**: `EnvironmentError`
- **Reason**: Issues with the environment setup such as incorrect working directory or missing environment variables.
- **Solution**: Verify the `workingDirectory` path and ensure all required environment variables are correctly set.


# cli/runNpmCommand

## Overview

The `cli/runNpmCommand` action facilitates the execution of npm commands within a specified working directory. This action is particularly useful in contexts where automated scripts need to run npm commands with specific parameters and within defined environments.

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

## Input Specifications

The input for this action is provided through the `with` object. Below are the required and optional parameters:

```yaml
- uses: cli/runNpmCommand
  with:
    args: "install"          # (required) The npm command arguments to execute.
    workingDirectory: "./"   # (optional) The working directory. Defaults to './'.
```
## Example Usage

Below is an example of how to configure the `cli/runNpmCommand` action within a YAML workflow:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
    
    - name: Run npm install
      uses: cli/runNpmCommand
      with:
        args: "install"
        workingDirectory: "./my-project"
```

This sample workflow checks out the repository and executes `npm install` within the `./my-project` directory.

### Input Validation Rules

1. **`args`**:
   - **Type**: `string`
   - **Description**: The arguments passed to the npm command.
   - **Required**: Yes
   - **Validation**: Must be a non-empty string.
   
2. **`workingDirectory`**:
   - **Type**: `string`
   - **Description**: The working directory for the npm command. Defaults to `./`.
   - **Required**: No
   - **Validation**: If provided, must be a valid directory path.
## Potential Errors for Troubleshooting

When using the `cli/runNpmCommand` action, users may encounter certain errors. Below are some common error scenarios, their reasons, and possible resolutions:

1. **`PrerequisiteError`**:
   - **Reason**: This error occurs when a required input parameter is missing or undefined.
   - **Possible Solutions**: Ensure that all required input parameters are provided and not null. For example, the `args` parameter must be a non-empty string.
   
2. **`ScriptExecutionError`**:
   - **Reason**: This error indicates that the script execution failed due to various reasons such as incorrect command syntax, missing dependencies, or environmental issues.
   - **Possible Solutions**:
     - Check the command syntax and ensure it is correct.
     - Verify that all necessary dependencies are installed.
     - Ensure that the environment variables and working directory settings are correctly configured.

3. **`ScriptTimeoutError`**:
   - **Reason**: This error occurs if the script execution exceeded the allowed time limit and was forcibly terminated.
   - **Possible Solutions**:
     - Increase the timeout parameter (if supported).
     - Optimize the npm script to run within the allowed time frame.


## Troubleshooting:
### Error message "Failed to run command"
Please check if the command exists in your system path and try to run this command manually in your working directory.


# cli/runDotnetCommand

The `cli/runDotnetCommand` action executes a Dotnet command with specified arguments. This document explains the action's functionality, the rules for input arguments, output specifications, and potential errors for troubleshooting.

## Overview

The `cli/runDotnetCommand` action leverages the Dotnet CLI to execute specified commands within a project, such as building or running applications. It supports configuring the working directory and specifying the path to the Dotnet executable. The executed command results are captured and can be directed to specified output variables.

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

## Input Arguments

The input arguments for this action are defined in the `with` object.

### Required Inputs

- **`args`** (string): The arguments passed to the Dotnet command.
  - *Example*: `args: "build"`

### Optional Inputs

- **`workingDirectory`** (string): The working directory where the command is executed. Defaults to `'./'`.
  - *Example*: `workingDirectory: "./src"`

- **`execPath`** (string): The path to the Dotnet executable. Defaults to the system path.
  - *Example*: `execPath: "/usr/local/bin/dotnet"`

### Example YAML Configuration

```yaml
uses: cli/runDotnetCommand
with:
  args: "build"
  workingDirectory: "./src"
  execPath: "/usr/local/bin/dotnet"
```
## Potential Errors and Troubleshooting

When executing the `cli/runDotnetCommand` action, several potential errors may occur. It's important to understand these errors and how to resolve them.

### Error: Missing Arguments

- **Class**: `PrerequisiteError`
- **Reason**: Required argument(s) are missing (`args` in this case).
- **Solution**: Ensure that all required arguments are provided in the `with` object.

### Error: Command Execution Failure

- **Class**: `ScriptTimeoutError` or `ScriptExecutionError`
- **Reason**: The Dotnet command failed to execute, could be due to a timeout or execution failure.
- **Solution**: 
  - Check the command arguments for correctness.
  - Verify that the specified `workingDirectory` and `execPath` are correct.
  - Ensure the Dotnet CLI is installed and accessible from the specified path.

### Example Troubleshooting Steps

1. **Check Inputs**: Verify that all required inputs are provided and correctly specified in your YAML configuration.
2. **View Logs**: Review the logs generated by the command to understand where it failed.
3. **Validate Environment**: Ensure the execution environment has the necessary dependencies installed, such as the correct version of the Dotnet CLI.

# azureAppService/zipDeploy

## Overview

The `azureFunctions/zipDeploy` action is designed to automate the process of uploading and deploying a project to Azure Functions using [the zip deploy feature](https://aka.ms/zip-deploy-to-app-services). It packages the specified distribution folder into a zip file and deploys it to the designated Azure Functions resource.

## What the Action Does

Upon execution, the action carries out the following steps:

1. **Input Validation**: Ensures that required inputs (`artifactFolder` and `resourceId`) are provided and valid.
2. **Packaging Files**: Packages the specified distribution folder (`artifactFolder`) into a zip file.
3. **Deployment**: Deploys the zip file to the specified Azure Functions resource (`resourceId`).
4. **Optional Dry Run**: If `dryRun` is set to `true`, the process terminates after packaging without actually deploying the files.
5. **Error Handling**: Captures and logs any errors that occur during the deployment process.

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

## Inputs

The inputs for the action are specified in the `with` object. The following inputs are required and optional parameters:

### Required Inputs

- `artifactFolder` (string): 
  - Description: Path to the distribution folder that contains the files to deploy.
  - Example: `/path/to/artifacts`

- `resourceId` (string):
  - Description: The resource id of the Azure App Service.
  - Example: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{appName}`

### Optional Inputs

- `workingDirectory` (string):
  - Description: The working directory. The deploy program will find ignore files and create the upload package file based on this directory. Defaults to `./`
  - Example: `/path/to/working-directory`

- `ignoreFile` (string):
  - Description: The path to the ignore file. Any files listed in this file will be ignored during upload. Defaults to ignoring nothing.
  - Example: `.deployignore`

- `dryRun` (boolean):
  - Description: If `true`, the action will only package the files to be deployed without actually deploying them. Defaults to `false`.
  - Example: `true`

- `outputZipFile` (string):
  - Description: The path to the packaged zip file. If not specified, the zip file will be saved to `workingDirectory/.deployment/deployment.zip`.
  - Example: `/path/to/output.zip`

### Example Usage in YAML

```yaml
uses: azureAppService/zipDeploy
with:
  artifactFolder: '/path/to/artifacts'
  resourceId: '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{appName}'
  workingDirectory: '/path/to/working-directory'
  ignoreFile: '.deployignore'
  dryRun: false
  outputZipFile: '/path/to/output.zip'
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

## Overview

The `azureFunctions/zipDeploy` action is designed to automate the process of uploading and deploying a project to Azure Functions using [the zip deploy feature](https://aka.ms/zip-deploy-to-azure-functions). It packages the specified distribution folder into a zip file and deploys it to the designated Azure Functions resource.

Upon execution, the action carries out the following steps:

1. **Input Validation**: Ensures that required inputs (`artifactFolder` and `resourceId`) are provided and valid.
2. **Packaging Files**: Packages the specified distribution folder (`artifactFolder`) into a zip file.
3. **Deployment**: Deploys the zip file to the specified Azure Functions resource (`resourceId`).
4. **Optional Dry Run**: If `dryRun` is set to `true`, the process terminates after packaging without actually deploying the files.
5. **Error Handling**: Captures and logs any errors that occur during the deployment process.

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
## Input Validation Rules

The action expects certain inputs through the `with` object. These inputs must adhere to specified rules:

- **Required Inputs:**
  - `artifactFolder`: Path to the folder containing the files to deploy. Must be a valid filesystem path. If your input value is a relative path, it is relative to the `workingDirectory`.
  - `resourceId`: The resource ID of the Azure Functions. This must be a valid Azure Resource ID. It is generated automatically after running the provision command. If you already have an Azure Function, you can find its resource ID in the Azure portal (see this [link](https://azurelessons.com/how-to-find-resource-id-in-azure-portal/) for more information).
  
- **Optional Inputs:**
  - `workingDirectory`: The directory to use as the base for relative paths. Defaults to the current directory (`"./"`).
  - `ignoreFile`: Path to a file listing the files to ignore during packaging. Accepts a valid filesystem path. This file can be utilized to exclude certain files or folders from the `artifactFolder`. Its syntax is similar to the Git's ignore.
  - `dryRun`: Boolean flag to indicate if the action should perform a dry run (i.e., package files without deploying). Defaults to `false`. This will help you verify that the packaging zip file is correct.
  - `outputZipFile`: Path to save the zipped file. Defaults to `workingDirectory/.deployment/deployment.zip`. This file will be reconstructed during deployment, reflecting all folders and files in your `artifactFolder`, and removing any non-existent files or folders.

### Example Inputs in YAML

```yaml
uses: azureFunctions/zipDeploy
with:
  artifactFolder: "dist"
  resourceId: "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{functionAppName}"
  workingDirectory: "./"
  ignoreFile: ".funcignore"
  dryRun: false
  outputZipFile: "output/deployment.zip"
```

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
## Overview
The `azureStorage/deploy` action uploads and deploys a project to Azure Storage Service. This action is designed for users who need to automate the deployment of their project files into an Azure Storage account.

The action handles reading input parameters (such as artifact folder and resource ID), processing the deployment, and managing error handling throughout the deployment process. It provides clear feedback and logs for the action execution, making it easy to troubleshoot and verify the deployment status.

## Syntax:
```
  - uses: azureStorage/deploy
    with:
      workingDirectory: ./src
      artifactFolder: . # Deploy base folder
      ignoreFile: ./.webappignore # Can be changed to any ignore file location, leave blank will ignore nothing
      resourceId: ${{BOT_AZURE_APP_SERVICE_RESOURCE_ID}} # The resource id of the cloud resource to be deployed to
```
## Input Validation Rules
The input parameters for the action are defined in the `with` object. Below are the parameters that the action accepts along with their validation rules:

- **artifactFolder** (required): Path to the distribution folder that contains the files to deploy.
  - Type: `string`
  - Description: The path must be valid and point to an existing folder containing the deployment artifacts.
  
- **resourceId** (required): The resource ID of the storage account.
  - Type: `string`
  - Description: A valid Azure resource ID for the target storage account.

- **workingDirectory** (optional): The working directory, deploy program will find ignore file and create upload package file based on this directory, default to `./`.
  - Type: `string`
  - Description: Default value is `./`. If provided, it should be a valid directory path.

- **ignoreFile** (optional): The path to the ignore file. Any files listed in this file will be ignored during upload.
  - Type: `string`
  - Description: Defaults to ignoring nothing. If provided, should be a valid path to an ignore file.
  
Example of a valid input in YAML format:
```yaml
with:
  artifactFolder: "./dist"
  resourceId: "/subscriptions/xxxx/resourceGroups/xxxx/providers/Microsoft.Storage/storageAccounts/xxxx"
  workingDirectory: "./"
  ignoreFile: ".deployignore”
```
## Output Specification
The action does not directly produce output as part of its execution. However, it manages internal logs and processes that can be utilized for debugging and status verification. Outputs of the action could be written to environment files which facilitate further steps in a CI/CD pipeline:

The following outputs can be stored:

- **DEPLOYMENT_STATUS**: The status of the deployment process.
  
The action uses the `writeToEnvironmentFile` object to specify the target output name and the environment variable name where the output value will be stored.

Example environment file entry:
```plaintext
DEPLOYMENT_STATUS=Success
```

## Troubleshooting:
### Error message: Failed to clear Azure Storage Account.
Please retry later or you can clear all files in your $web container and retry the deployment action.

### Error message: Failed to upload local path xxxx to Azure Storage Account.
Please retry this action later.

### 1. `BaseComponentInnerError`
- **Reason**: Custom operational errors within the deployment component.
- **Solution**: Ensure that the input parameters are correct and valid. Check the logs for detailed error messages to understand the specific issue.

### 2. `SystemError`
- **Reason**: General system errors including network issues, filesystem problems, or other operational interruptions.
- **Solution**: Retry the deployment operation. Verify network connectivity and the availability of the target storage account. Ensure the server has sufficient permissions and resources.

### 3. `UserError`
- **Reason**: Errors due to invalid user input or configuration issues.
- **Solution**: Double-check the provided inputs, especially `artifactFolder` and `resourceId`. Ensure that all required parameters are correct and the paths are valid.

### 4. `UnknownError`
- **Reason**: Errors that are not recognized by the system, which may stem from unexpected conditions or unknown issues.
- **Solution**: Consult the detailed error logs provided in the action execution output. Gather the error details and consider consulting Azure support if the issue persists.


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

The `botFramework/create` action is designed to seamlessly create or update bot registrations on [dev.botframework.com](https://dev.botframework.com/bots). . The action performs the following tasks:

1. Validates input parameters.
2. Checks the existence of an existing bot registration using the provided `botId`.
3. Creates a new bot registration if it does not exist.
4. Updates any existing bot registration with provided details.
5. Returns execution results and summaries.

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

## Input Parameters

The input arguments for the `botFramework/create` action are defined in the `with` object. Below are the required and optional fields:

### Required Fields

- **botId** (string): The Microsoft Entra app client ID of the bot.
- **name** (string): The name of the bot.
- **messagingEndpoint** (string): The messaging endpoint of the bot.

### Optional Fields

- **description** (string): The long description of the bot.
- **iconUrl** (string): The icon URL of the bot.
- **channels** (array): The channels to be enabled for the bot. Supported channels are `MsTeamsChannel` and `M365ExtensionsChannel`.

### Example Input

```yaml
with:
  botId: "123e4567-e89b-12d3-a456-426614174000"
  name: "SampleBot"
  messagingEndpoint: "https://example.com/api/messages"
  description: "This is a sample bot."
  iconUrl: "https://example.com/icon.png"
  channels:
    - name: "MicrosoftTeams"
      callingWebhook: "https://example.com/webhook"
    - name: "M365Extensions"
```

## Input Validation Rules

The validation of input parameters includes:
- **botId**: Must be a non-empty string and a valid UUID.
- **name**: Must be a non-empty string.
- **messagingEndpoint**: Must be a valid URL string.
- **description**: Must be a string if provided.
- **iconUrl**: Must be a string if provided.
- **channels**: Must be an array if provided; each item must have a name (must be either `MicrosoftTeams` or `M365Extensions`), and if a calling webhook is specified for `MicrosoftTeams`, it must be a string.

## Output Specification

The output of the action execution is defined in the `writeToEnvironmentFile` object. The key is the target output name, and the value is the environment variable where the output will be stored.

## Potential Errors for Troubleshooting

### Error Classes and Reasons

1. **InvalidActionInputError**
   - **Reason:** Invalid input parameters.
   - **Solution:** Ensure all required fields are provided and are of the correct data type.

2. **InvalidBotIdUserError**
   - **Reason:** `botId` is not a valid UUID.
   - **Solution:** Provide a valid UUID for `botId`.

3. **BotFrameworkNotAllowedToAcquireTokenError**
   - **Reason:** Unauthorized access while acquiring token.
   - **Solution:** Check permissions and ensure the correct credentials are used.

4. **BotFrameworkForbiddenResultError**
   - **Reason:** Forbidden access to Bot Framework resources.
   - **Solution:** Verify permissions and API scope.

5. **BotFrameworkConflictResultError**
   - **Reason:** Too many requests or conflicting operations.
   - **Solution:** Retry the operation after some time.

6. **ProvisionError**
   - **Reason:** Failure during bot creation.
   - **Solution:** Review the error message and ensure all inputs and API details are correct.

7. **ConfigUpdatingError**
   - **Reason:** Failure during bot update.
   - **Solution:** Confirm the bot configuration details and retry the update.

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

## Overview

The **`devTool/install`** action is designed to automate the installation of various development tools required for efficient software development. This action includes the following key sub-tasks:

1. Resolving local SSL certificates.
2. Installing Azure Functions Core Tools.
3. Installing .NET SDK.
4. Installing Teams App Test Tool.

The guide provides a comprehensive overview, input validation rules, output specifications, and potential errors along with troubleshooting tips.

---

## What the Action Does

`ToolsInstallDriver.execute` function manages the overall execution flow, encompassing the following primary tasks:

1. **Resolve Local Certificates:**
   - Install and optionally trust local SSL certificates.

2. **Resolve Function Core Tools:**
   - Install Azure Functions Core Tools and manage versioning.

3. **Resolve .NET SDK:**
   - Install .NET SDK based on configuration.

4. **Resolve Test Tool:**
   - Install Teams App Test Tool and manage versioning.

---

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

## Input Validation Rules

Inputs should be specified under the `with` parameter in a YAML format. Each input must comply with the following validation rules.

### Example Input (YAML)

```yaml
uses: devTool/install
with:
  devCert:
    trust: true
  func:
    version: "3.x"
    symlinkDir: "/path/to/symlink"
  dotnet: true
  testTool:
    version: "1.0.0"
    symlinkDir: "/path/to/symlink"
```

### Input Parameters

- **devCert**:
  - **trust** (required, boolean): Indicates whether to trust the SSL certificate.

- **func**:
  - **version** (required, string/integer): Specifies the semantic versioning for Azure Functions Core Tools.
  - **symlinkDir** (optional, string): Directory path for symlinked function binaries.

- **dotnet** (optional, boolean): Determines if the .NET SDK should be installed.

- **testTool**:
  - **version** (required, string/integer): Specifies the version number of Teams App Test Tool.
  - **symlinkDir** (required, string): Directory path for symlinked binaries for the Teams App Test Tool.

---

## Output Specification

The results of the action's execution will be written as environment variables. These are specified in the `writeToEnvironmentFile` object.

### Example Output Configuration (YAML)

```yaml
writeToEnvironmentFile:
  sslCertFile: SSL_CERTIFICATE_FILE
  sslKeyFile: SSL_KEY_FILE
  funcPath: FUNCTIONS_CORE_TOOLS_PATH
  dotnetPath: DOTNET_PATH
  testToolPath: TEST_TOOL_PATH
```

### Output Variables

- **sslCertFile**: Path to the SSL certificate file.
- **sslKeyFile**: Path to the SSL key file.
- **funcPath**: Path to the Azure Functions Core Tools binary.
- **dotnetPath**: Path to the .NET binary.
- **testToolPath**: Path to the Teams App Test Tool binary.

---

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

## Overview

The `arm/deploy` action is responsible for creating Azure resources using referenced Bicep/JSON files. The outputs from these templates are stored in the current Teams Toolkit environment. This action helps automate Azure Resource Manager (ARM) template deployment, making it simpler to manage and scale Azure resources.

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

## Input Validation Rules

The input arguments for this action are defined in the `with` object. The input fields required for the action and their validation rules are:

- `subscriptionId`: The subscription ID to deploy to. It should be a valid UUID.
- `resourceGroupName`: The resource group name to deploy to. It must be a non-empty string.
- `templates`: An array of templates to deploy. Each template should have:
  - `deploymentName`: The name of the ARM deployment. It must be a non-empty string.
  - `path`: The relative path to the ARM template. Both Bicep and JSON formats are supported.
  - `parameters` (optional): The relative path to the ARM parameters file. Teams Toolkit will expand environment variables in the parameters file.

### Example Input

```yaml
with:
  subscriptionId: "your-subscription-id"
  resourceGroupName: "your-resource-group"
  bicepCliVersion: "0.4.1008" # optional
  templates:
    - deploymentName: "template1-deployment"
      path: "./templates/template1.bicep"
      parameters: "./parameters/template1-parameters.json" # optional
    - deploymentName: "template2-deployment"
      path: "./templates/template2.json"
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

Below are the potential errors encountered during the execution of the `arm/deploy` action, along with possible reasons and suggested solutions:

### InvalidAzureCredentialError

- **Reason:** Azure credentials are required but failed to obtain.
- **Solution:** Ensure that the Azure account is authenticated and that the credentials are correctly configured.

### CompileBicepError

- **Reason:** An error occurred while compiling the Bicep file to JSON.
- **Solution:** Ensure that the Bicep file syntax is correct and that the Bicep CLI is correctly installed.

### MissingEnvironmentVariablesError

- **Reason:** Required environment variables are missing in the parameters file.
- **Solution:** Verify that all necessary environment variables are defined and accessible.

### ConvertArmOutputError

- **Reason:** The output key from the ARM deployment result has a naming conflict.
- **Solution:** Check the ARM deployment output for naming collisions and adjust the template or action accordingly.

### DeployArmError

- **Reason:** An error occurred during the ARM deployment.
- **Solution:** Review the ARM deployment logs to identify the specifics of the deployment failure.

### InstallSoftwareError

- **Reason:** The installation of the Bicep CLI failed.
- **Solution:** Check the network connection and permissions required to download and install the Bicep CLI.

### UserError / SystemError

- **Reason:** Broad categories for user-generated or system-generated errors.
- **Solution:** Review the specific error message to determine whether it's an issue with the input values or a broader system issue.


# botAadApp/create

## Overview

The `botAadApp/create` action is designed to create a new or reuse an existing Microsoft Entra application specifically for a bot. This action simplifies the process of setting up the necessary Microsoft Entra resources for bot authentication, which are essential for integrating bots with various Microsoft services.

## Syntax:
```yaml
  - uses: botAadApp/create # Creates a new or reuses an existing Microsoft Entra application for bot.
    with:
      name: {{appName}}-${{TEAMSFX_ENV}} # The Microsoft Entra application's display name
    writeToEnvironmentFile:
      botId: BOT_ID # The Microsoft Entra application's client id created for bot.
      botPassword: SECRET_BOT_PASSWORD # The Microsoft Entra application's client secret created for bot. 
```
## Input Validation Rules

The input arguments for this action are defined within the `with` object. Below are the expected input parameters and their respective validation rules:

- **name**: `string` (required)
  - Description: The user-facing display name for the Microsoft Entra application.
  - Validation: Must be a non-empty string, maximum length of 120 characters.
  - Example:

  ```yaml
  with:
    name: "MyBotEntraApp"
  ```
## Output Specification

The outputs generated by the action execution are defined in the `writeToEnvironmentFile` object. Here are the expected outputs:

- **botId**: Stores the client (application) ID of the created Microsoft Entra application.
- **botPassword**: Stores the client secret of the created Microsoft Entra application.

These outputs are returned as environment variables, and their values are essential for further bot operations.

## Example Configuration

```yaml
botAadApp/create:
  with:
    name: "MyBotEntraApp"

  writeToEnvironmentFile:
    botId: "BOT_ID"
    botPassword: "BOT_PASSWORD"
```

## Potential Errors for Troubleshooting

When executing this action, several potential errors might arise. Below are the error classes, reasons, and possible solutions:

- **InvalidActionInputError**:
  - **Reason**: Occurs if the input parameters do not meet the validation criteria (e.g., missing `name` or name length exceeding 120 characters).
  - **Solution**: Ensure the `name` parameter is provided and is a valid string, not exceeding 120 characters.

- **OutputEnvironmentVariableUndefinedError**:
  - **Reason**: Triggered when the expected output environment variables are not defined.
  - **Solution**: Check the `writeToEnvironmentFile` configuration for correct environment variable entries (`botId` and `botPassword`).

- **UnexpectedEmptyBotPasswordError**:
  - **Reason**: Raised if a bot ID exists but its corresponding password is missing.
  - **Solution**: Ensure that both the bot ID and bot password are correctly specified or generated.

- **UserError** or **SystemError**:
  - **Reason**: Generic errors related to user or system malfunctions.
  - **Solution**: Inspect the error message to determine the specific issue and consider retrying or adjusting inputs.

- **HttpClientError**:
  - **Reason**: HTTP client-side errors (status code 400-499) during communication with the Microsoft Graph API.
  - **Solution**: Verify network connectivity and ensure correct API permissions and inputs.

- **HttpServerError**:
  - **Reason**: HTTP server-side errors (status code 500-599) encountered during API calls.
  - **Solution**: Wait for a while and retry the operation as these errors are usually temporary.

- **CredentialInvalidLifetimeError**:
  - **Reason**: The client secret's lifetime does not comply with the application's policy.
  - **Solution**: Adjust the client secret's expiration date according to the policy guidelines.

- **ClientSecretNotAllowedError**:
  - **Reason**: The client secret type is not permitted as per the application's policy.
  - **Solution**: Use an allowed client secret type.


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
 
## Overview

The `apiKey/register` action is designed to create a new API key for a Teams app. This process involves:

1. Validating the provided input parameters.
2. Connecting to the Teams Developer Portal to register the API key.
3. Storing the registration ID of the created API key into an environment variable for future use.

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
## Input Validation Rules

The input arguments are provided within the `with` object. Here's the schema for the required and optional parameters:

```yaml
with:
  name: "example-api-key"              # Required, string: The name of API key.
  appId: "your-app-id"                 # Required, string: The app ID of Teams app.
  apiSpecPath: "path/to/api/spec.yaml" # Required, string: The path of API specification file.
  primaryClientSecret: "abcd1234"      # Optional, string: Primary client secret of API key. Length must be between 10 and 128.
  secondaryClientSecret: "efgh5678"    # Optional, string: Secondary client secret of API key. Length must be between 10 and 128.
  applicableToApps: "AnyApp"           # Optional, string: Specifies which app can access the API key. Default is "AnyApp".
  targetAudience: "AnyTenant"          # Optional, string: Specifies which tenant can access the API key. Default is "AnyTenant".
```

### Examples

**Valid Input Example:**

```yaml
with:
  name: "example-api-key"
  appId: "12345-abcdef-67890"
  apiSpecPath: "specs/apiSpec.yaml"
  primaryClientSecret: "Nkjdh36Gfhshs"
  secondaryClientSecret: "Wkej378Hgdhsh29"
  applicableToApps: "SpecificApp"
  targetAudience: "HomeTenant"
```

**Invalid Input Example:**

```yaml
with:
  name: ""  # Invalid, name is required and cannot be empty
  appId: "12345-abcdef-67890"
  apiSpecPath: "specs/apiSpec.yaml"
  primaryClientSecret: "short"  # Invalid, primaryClientSecret must be at least 10 characters long
  applicableToApps: "InvalidValue"  # Invalid, must be "SpecificApp" or "AnyApp"
```

**Note:** All required fields (`name`, `appId`, `apiSpecPath`) must be provided and follow the correct formats and constraints.

## Output Specification

The output of the action execution is written to the environment file specified in the `writeToEnvironmentFile` object. 

Here is the schema for the output specification:

```yaml
writeToEnvironmentFile:
  registrationId: "API_KEY_REGISTRATION_ID"  # Required: The registration ID of the created API key.
```

## Potential Errors for Troubleshooting

When executing the `apiKey/register` action, several errors might occur. Below is a list of potential errors, their descriptions, and suggested solutions:

### 1. `InvalidActionInputError`

- **Reason:** One or more parameters in the input are invalid.
- **Solution:** Ensure all required parameters are provided and follow the correct formats and constraints as defined in the input validation rules.

### 2. `ApiKeyClientSecretInvalidError`

- **Reason:** The provided client secret(s) do not meet the length requirements (10 to 128 characters).
- **Solution:** Verify that the `primaryClientSecret` and/or `secondaryClientSecret` are between 10 and 128 characters long.

### 3. `ApiKeyDomainInvalidError`

- **Reason:** The number of domains associated with the API key exceeds the maximum allowed limit.
- **Solution:** Ensure the number of domains specified in the API specification file does not exceed the allowed limit.

### 4. `ApiKeyFailedToGetDomainError`

- **Reason:** No valid domain was retrieved from the API specification file.
- **Solution:** Verify the API specification file for correctness and ensure it includes valid domains.

### 5. `OutputEnvironmentVariableUndefinedError`

- **Reason:** The output environment variable mapping is not defined.
- **Solution:** Ensure the `writeToEnvironmentFile` object contains a valid `registrationId` key with the corresponding environment variable name.

### 6. `SystemError` or `UserError`

- **Reason:** An internal error occurred during the action execution.
- **Solution:** Check the detailed error message for troubleshooting steps. Common causes might include network issues, authentication problems, or issues with the Teams Developer Portal API.

# apiKey/update
This action will update an API key in Developer Portal for authentication of API based message extension.
 
## Overview
The `apiKey/update` action is used to update an existing API key's properties within the Teams application ecosystem. The action validates the provided input parameters, checks for differences from the current API key properties, and updates the API key if necessary.

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
## Input Validation Rules
The `apiKey/update` action accepts a set of input parameters defined within a `with` object. These parameters and their validation rules are as follows:

- **name**:
  - **Type**: `string`
  - **Description**: The name of the API key.
  - **Validation**: Must be a non-empty string and have a maximum length of 128 characters.

- **appId**:
  - **Type**: `string`
  - **Description**: The app ID of the Teams app.
  - **Validation**: Must be a non-empty string.

- **apiSpecPath**:
  - **Type**: `string`
  - **Description**: The path of the API specification file.
  - **Validation**: Must be a non-empty string.

- **registrationId**:
  - **Type**: `string`
  - **Description**: The registration ID of the API key.
  - **Validation**: Must be a non-empty string.

- **applicableToApps** (optional):
  - **Type**: `string`
  - **Description**: Determines which app can access the API key. Values can be `"SpecificApp"` or `"AnyApp"`. Default is `"AnyApp"`.
  - **Validation**: Must be one of the specified enum values.

- **targetAudience** (optional):
  - **Type**: `string`
  - **Description**: Determines which tenant can access the API key. Values can be `"HomeTenant"` or `"AnyTenant"`. Default is `"AnyTenant"`.
  - **Validation**: Must be one of the specified enum values.

### Example Input
```yaml
with:
  name: "MyUpdatedApiKey"
  appId: "12345678-1234-1234-1234-1234567890ab"
  apiSpecPath: "./specs/api.json"
  registrationId: "9abcdef0-1234-5678-abcd-ef0123456789"
  applicableToApps: "SpecificApp"
  targetAudience: "HomeTenant"
```

## Output Specification
The outputs of the `apiKey/update` action are defined in the `writeToEnvironmentFile` object. This object specifies where to write the results of the update.

### Example Output
- **result**:
  - **Description**: This could store the success message or any output data post execution.
  - **Type**: `string`
  - **Example**: "API Key updated successfully."

## Potential Errors for Troubleshooting

### UserError
- **Reason**: This type of error occurs when there is a mistake in the input parameters provided by the user.
- **Possible Solutions**: Verify that all required parameters are correct and meet the validation rules.

### SystemError
- **Reason**: This error type occurs due to system issues, such as network problems or API failures.
- **Possible Solutions**: Ensure that the system is functioning correctly, and retry the action after some time. Check network connections and API availability.

### ApiKeyNameTooLongError
- **Reason**: The provided API key name exceeds the length limit.
- **Possible Solutions**: Ensure the `name` parameter is no longer than 128 characters.

### ApiKeyDomainInvalidError
- **Reason**: The provided domain list exceeds the maximum allowed domains or is empty.
- **Possible Solutions**: Ensure that the domain list retrieved from the API spec file is correct and within the acceptable limits.

### InvalidActionInputError
- **Reason**: One or more input parameters are invalid.
- **Possible Solutions**: Review the input parameters and ensure they meet the validation criteria specified.

### ApiKeyFailedToGetDomainError
- **Reason**: The action failed to retrieve valid domains from the API specification.
- **Possible Solutions**: Check the API specification file to ensure it contains valid bearer token authentication schemes and the correct server configurations.


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