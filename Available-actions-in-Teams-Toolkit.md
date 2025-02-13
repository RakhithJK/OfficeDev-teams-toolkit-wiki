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
The `teamsApp/create` action automates the process of creating a new Microsoft Teams app using the Teams Developer Portal. It can either create a new app if an existing Teams App ID is not found or ensure that the required Teams app exists for a given Teams App ID.

## Syntax:
```
  - uses: teamsApp/create
    with:
      name: YOUR-APP-NAME-${{TEAMSFX_ENV}} # TEAMSFX_ENV is the environment variable defined in env/.env.<environment> file, used to differentiate Teams app in Teams Developer Portal
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      teamsAppId: TEAMS_APP_ID
```
## Input Validation Rules
The input arguments for this action are defined within the `with` object. All inputs must be provided in a structured YAML format.

### Required Inputs
- **name**: The name of the Teams app to be created.

```yaml
with:
  name: "Sample Teams App"
```

### Optional Inputs
- **uses**: The action command, should be `teamsApp/create`.
- **env**: Environment variables which might be required for the action.

```yaml
uses: "teamsApp/create"
env:
  CUSTOM_ENV_VAR: "some-value"
  
with:
  name: "Sample Teams App"
```

## Output Specification
The output from this action is written to the environment file as specified in the `writeToEnvironmentFile` object. The action primarily sets the following outputs:

- **teamsAppId**: The ID of the created or verified Teams app.
- **teamsAppTenantId**: The tenant ID associated with the Teams app.

Example configuration in YAML:

```yaml
writeToEnvironmentFile:
  teamsAppId: "TEAMS_APP_ID"
  teamsAppTenantId: "TEAMS_APP_TENANT_ID"
```
## Potential Errors for Troubleshooting
The Teams App Create action can encounter several errors during its execution. Below is a list of potential errors, their causes, and possible solutions:

### User Errors
#### InvalidActionInputError
- **Reason**: The required input parameters are missing or invalid.
- **Solution**: Ensure that the `name` parameter is provided in the input YAML.

#### TeamsAppCreateConflictError
- **Reason**: There is a conflict with an existing app registration.
- **Solution**: Verify if the app already exists and handle the conflict, or use a different app name.

#### TeamsAppCreateConflictWithPublishedAppError
- **Reason**: The provided app ID conflicts with an existing published app.
- **Solution**: Avoid using an app ID that is already associated with a published Teams app.

#### InvalidTeamsAppIdError
- **Reason**: The given App ID must be a valid GUID.
- **Solution**: Ensure the App ID follows the GUID format.

### System Errors
#### TeamsAppCreateFailedError
- **Reason**: General failure during the Teams app creation.
- **Solution**: Utilize the provided help link and investigate the cause further. Ensure all dependencies and permissions are correctly configured.


# teamsApp/update
Apply the Teams app manifest to an existing Teams app in Teams Developer Portal. Will use the app id in manifest.json file to determine which Teams app to update.

### What This Action Does:
- Reads the app package from the provided path.
- Extracts the `manifest.json` file from the app package to retrieve the Teams app ID.
- Validates that the Teams app ID is correct and the app exists in the Teams Developer Portal.
- Updates the app in Teams Developer Portal with the new contents from the app package.


## Syntax:
```
  - uses: teamsApp/update # 
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
```

## Input Validation Rules

The action expects input parameters defined under the `with` object. Below is the expected structure and validation rules for the inputs.

### Parameters:

| Parameter      | Type   | Description                        | Requirement                           |
|----------------|--------|------------------------------------|---------------------------------------|
| `appPackagePath` | String | Path to the Teams app package file. | Required |

### Example Input:

```yaml
uses: teamsApp/update
with:
  appPackagePath: "./path/to/teamsAppPackage.zip"
```

#### Rules:
- `appPackagePath` must not be null or empty and must point to an existing file.
- The `manifest.json` inside the app package must contain a valid Teams app ID (UUID format).

### Detailed Example with Environment Variables:

```yaml
uses: teamsApp/update
env:
  M365_CLIENT_ID: <your-client-id>
  M365_CLIENT_SECRET: <your-client-secret>
  M365_TENANT_ID: <your-tenant-id>

with:
  appPackagePath: "./path/to/teamsAppPackage.zip"
```

## Output Specification

Upon successful execution, the action will output specific details to environment variables. These details help to confirm and log the update process.

### Output:

| Key                        | Environment Variable        | Description                           |
|----------------------------|-----------------------------|---------------------------------------|
| `teamsAppTenantId`         | teamsAppTenantId            | The tenant ID where the Teams app resides. |
| `teamsAppUpdateTime`       | teamsAppUpdateTime          | The timestamp when the Teams app was last updated. |

## Potential Errors for Troubleshooting

When executing this action, you may encounter several types of errors. Here will enumerate possible error classes, reasons, and suggested solutions.

### Error Classes, Reasons, and Solutions:

#### 1. **FileNotFoundError**
- **Reason**: The specified app package path does not exist.
- **Solution**: Ensure that the `appPackagePath` is correct and points to an existing file.

#### 2. **InvalidTeamsAppIdError**
- **Reason**: The Teams app ID extracted from the `manifest.json` is not a valid UUID.
- **Solution**: Validate the `manifest.json` file within your app package to ensure the ID is correctly formatted as a UUID.

#### 3. **TeamsAppNotExistsError**
- **Reason**: The Teams app specified by the extracted ID does not exist in the Teams Developer Portal.
- **Solution**: Double-check the Teams app ID in the `manifest.json` to make sure you're targeting the correct app.

#### 4. **TeamsAppUpdateFailedError**
- **Reason**: General error occurred while attempting to update the Teams app.
- **Solution**: Review the error message and check any logs for more details; you may need to troubleshoot based on additional context given by the error message.

#### 5. **AccessTokenError**
- **Reason**: Failed to obtain an access token for the Microsoft 365 environment.
- **Solution**: Validate your environment variables (`M365_CLIENT_ID`, `M365_CLIENT_SECRET`, `M365_TENANT_ID`) for correctness and retry.


# teamsApp/validateManifest
The action `teamsApp/validateManifest` is designed to validate the Microsoft Teams app manifest file. It ensures that the manifest conforms to the predefined schema, checks for any errors or inconsistencies, and performs an additional validation if copilot extensions are present. This action is part of the broader set of tools provided to streamline the development and deployment of Teams applications.

## Action Description

The `teamsApp/validateManifest` action will:
- Validate the provided Teams app manifest against the relevant schema.
- Check for the presence of copilot extensions and their valid schema.
- Use environment variables referenced within the manifest for validation.
- Report detailed validation errors and result summaries.

## Syntax:
```
  - uses: teamsApp/validate
    with:
      manifestPath: ./appPackage/manifest.json # Required. Path to Teams app manifest file
```

## Input Specifications

The inputs for this action are provided through the `with` object. The only required input is the `manifestPath`.

### Required Inputs

- `manifestPath`: Path to the Teams app manifest file.

**Example Input (YAML Format):**
```yaml
uses: teamsApp/validateManifest
with:
  manifestPath: "./path/to/teams-app-manifest.json"
```
## Potential Errors for Troubleshooting

When performing the `teamsApp/validateManifest` action, various errors might occur. Below are the common errors, their reasons, and possible solutions:

### 1. `ValidationFailedError`

- **Error Reason**: The manifest validation failed due to schema mismatches or errors in the manifest content.
- **Possible Solutions**:
  - Ensure that the manifest file conforms to the defined schema.
  - Verify the JSON syntax of the manifest.
  - Cross-check any URLs or required fields within the manifest for accuracy.

### 2. `InvalidActionInputError`

- **Error Reason**: The input provided to the action is either missing or invalid.
- **Possible Solutions**:
  - Ensure the `manifestPath` is provided.
  - Verify the path specified in `manifestPath` is correct and points to a valid file.

### 3. `FileNotFoundError`

- **Error Reason**: The manifest file specified in `manifestPath` does not exist.
- **Possible Solutions**:
  - Verify the file path provided.
  - Ensure the file is not deleted or moved from the specified location.

### 4. `JSONSyntaxError`

- **Error Reason**: The manifest file contains invalid JSON.
- **Possible Solutions**:
  - Validate the JSON syntax using tools like `jsonlint`.
  - Ensure proper formatting of the manifest file.

### 5. `MissingEnvironmentVariablesError`

- **Error Reason**: The manifest references environment variables that are not set.
- **Possible Solutions**:
  - Define all required environment variables.
  - Ensure the variable names are correct and match those in the manifest.


# teamsApp/validateAppPackage

The `teamsApp/validateAppPackage` action validates a Microsoft Teams app package file against a set of predefined validation rules. It ensures that the app package conforms to the required standards and formats before it is submitted to the Teams Developer Portal.

## Syntax:
```
  - uses: teamsApp/validateAppPackage
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
```
## Input Validation Rules

The input parameters for this action are defined in the `with` object. The required and optional parameters are as follows:

- `appPackagePath` (required): The path to the zipped Teams app package file.

### Example Input

```yaml
uses: teamsApp/validateAppPackage
with:
  appPackagePath: 'path/to/your/teams/app/package.zip'
```

## Output Specification

The outputs of the action's execution are defined in the `writeToEnvironmentFile` object. The key is the target output name, and the value is the environment variable name to store the output value. Based on the source code, here are the potential outputs:

- `validationSummary`: A summary of the validation results, including errors, warnings, and passes.
- `validationResult`: Detailed results of the validation, including individual errors and warnings.

## Potential Errors for Troubleshooting

Here are some common errors that might occur during the execution of the `teamsApp/validateAppPackage` action, along with their reasons and potential solutions:

1. **InvalidActionInputError**
   - **Reason**: The required input parameter `appPackagePath` is missing or invalid.
   - **Solution**: Ensure that the `appPackagePath` parameter is provided and points to the correct path of the zipped Teams app package file. Refer to the [Action Input Documentation](https://aka.ms/teamsfx-actions/teamsapp-validate) for more details.

2. **FileNotFoundError**
   - **Reason**: The specified app package file could not be found at the given path.
   - **Solution**: Verify that the `appPackagePath` is correct and that the file exists at the specified location.

3. **ValidationFailedError**
   - **Reason**: The app package failed validation due to errors.
   - **Solution**: Check the error messages provided in the validation summary. Review and correct the issues mentioned in the error messages.

4. **AppStudioValidationError**
   - **Reason**: An error occurred while communicating with the Teams Developer Portal API for validation.
   - **Solution**: Ensure that you have a valid Microsoft 365 token and that you have network connectivity. Retry the validation process. If the issue persists, refer to the API documentation or support.


# teamsApp/validateWithTestCases
The `teamsApp/validateWithTestCases` action performs asynchronous validation tests on a Microsoft Teams app package file. This process ensures that your app meets all required standards and guidelines for Microsoft Teams apps. 
For more details, refer to the [documentation](https://aka.ms/teamsfx-actions/teamsapp-validate).

## Functionality
Upon execution, this action:
1. Validates the provided input parameters.
2. Checks if the specified app package file exists.
3. Reads and processes the manifest from the app package.
4. Initiates an application validation request with the Azure App Studio.
5. Provides periodic updates and final results of the validation process.

This validation ensures that the uploaded Teams app package file conforms to Microsoft Teams' guidelines and standards.

## Syntax:
```
  - uses: teamsApp/validateWithTestCases
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
      showMessage: true # Optional. Show message or not.
      showProgressBar: true # Optional. Show progress bar or not.
```

## Input Validation
The input parameters for this action are defined under the `with` object. All parameters are required unless otherwise specified.

### Parameters
- `appPackagePath` (string): **Required**. The path to the zipped Teams app package file.
- `showMessage` (boolean): *Optional*. Whether to display messages during the validation process.
- `showProgressBar` (boolean): *Optional*. Whether to show a progress bar during the validation process.

### Example Input
```yaml
with:
  appPackagePath: "./path/to/teamsApp.zip"
  showMessage: true
  showProgressBar: true
```
## Potential Errors
The action might encounter several errors which have been categorized as follows:

### 1. Input Errors
- **Error Class**: InvalidActionInputError
- **Reason**: Missing required input parameters (`appPackagePath`).
- **Solution**: Ensure all required parameters are correctly specified in the input. Refer to the [documentation](https://aka.ms/teamsfx-actions/teamsapp-validate-test-cases) for more details.

### 2. File Errors
- **Error Class**: FileNotFoundError
- **Reason**: The specified app package file does not exist at the given path.
- **Solution**: Verify that the file path is correct and the app package file exists.

### 3. App Studio Errors
- **Error Class**: AppStudioError
- **Reason**: Issues related to App Studio, such as token retrieval failures or network issues.
- **Solution**: Ensure your App Studio token is valid and accessible. Check network connection and retry.

### 4. Validation Status Errors
- **Error Class**: AsyncAppValidationError
- **Reason**: Errors in the validation status such as incomplete or aborted validations.
- **Solution**: Check the status at the provided URL. If a current validation is in progress, wait until it completes before submitting a new request.


# teamsApp/zipAppPackage
The `teamsApp/zipAppPackage` action is designed to facilitate the process of packaging a Microsoft Teams app. It achieves this by rendering the Teams app manifest template with environment variables, and then zipping the manifest file along with two icons (color and outline icons). This action ensures a streamlined process for generating the required app package for deployment in Teams.

## Syntax:
version 1.7 or higher:
```
  - uses: teamsApp/zipAppPackage
    with:
      manifestPath: ./appPackage/manifest.json # Required. Relative path to this file. This is the path for Teams app manifest file.
      outputZipPath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
      outputFolder: ./appPackage/build # Required. Relative path to this file. This is the folder where all resolved manifest(s) will be placed.
```
version < 1.6
```
  - uses: teamsApp/zipAppPackage
    with:
      manifestPath: ./appPackage/manifest.json # Required. Relative path to this file. This is the path for Teams app manifest file.
      outputZipPath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
      outputJsonPath: ./appPackage/build/manifest.${{TEAMSFX_ENV}}.json # Required. Relative path to this file. This is the path for built manifest json file.
```

## Input Parameters

The input parameters for this action are passed within the `with` object. 

### Required Inputs

- **manifestPath** (`string`): Path to the Teams app manifest file.
- **outputZipPath** (`string`): Path where the output ZIP package will be created.
- **If version >= 1.7, outputFolder** (`string`): Path of the folder where all resolved manifest(s) will be placed.
- **If version < 1.7, outputJsonPath** (`string`): Path where the output manifest JSON file will be generated.

### Example Input
version 1.7 or higher:
```yaml
- uses: teamsApp/zipAppPackage
  with:
    manifestPath: "path/to/manifest.json"
    outputZipPath: "path/to/output.zip"
    outputFolder: "path/to/outputFolder"
```

version < 1.7
```yaml
- uses: teamsApp/zipAppPackage
  with:
    manifestPath: "path/to/manifest.json"
    outputZipPath: "path/to/output.zip"
    outputJsonPath: "path/to/output.json"
```

## Process Flow and Detailed Execution

1. **Input Validation**: Before proceeding, the action validates the provided input parameters to ensure that all required inputs (`manifestPath`, `outputZipPath`, and `outputJsonPath`) are present. If any of these parameters are missing, the action throws an `InvalidActionInputError`.

2. **Path Normalization**: The paths provided in the inputs are normalized to absolute paths to ensure that the files can be accessed correctly.

3. **Manifest File Processing**: 
    - The manifest file specified by `manifestPath` is read and processed.
    - Environment variables within the manifest file are resolved.

4. **Icon Files Verification**:
    - The action checks the existence of the icon files (color and outline) as specified in the manifest file. If these files are missing or located outside the allowed directory, corresponding errors (`FileNotFoundError` or `InvalidFileOutsideOfTheDirectotryError`) are thrown. 

5. **Localization Files Verification**:
    - If localization files are specified in the manifest, their existence is verified similarly. Missing files will result in appropriate exceptions.

6. **Packaging**:
    - The manifest file and the icon files are added to a ZIP package.
    - If present, additional localization files, API specifications, Adaptive card templates, and plugins are included in the ZIP package.

7. **Output Generation**:
    - The final ZIP package is written to the `outputZipPath`.
    - The processed manifest JSON is written to the `outputJsonPath`.

8. **Summary and Logs**:
    - Once the process is complete, relevant summaries and success messages are logged for user reference.
## Potential Errors and Troubleshooting

### InvalidActionInputError
- **Reason**: One or more required input parameters are missing.
- **Solution**: Ensure that all required parameters (`manifestPath`, `outputZipPath`, and `outputJsonPath`) are specified.

### FileNotFoundError
- **Reason**: The specified file (manifest, icon, localization, or plugin file) does not exist at the expected path.
- **Solution**: Verify that all specified files exist at the given paths and adjust the paths if necessary.

### InvalidFileOutsideOfTheDirectotryError
- **Reason**: One of the specified files is outside the allowed directory scope.
- **Solution**: Ensure that all file paths are within the appropriate directory structure relative to the manifest file.

### JSONSyntaxError
- **Reason**: The manifest or plugin file has incorrect JSON syntax.
- **Solution**: Validate the JSON syntax of the manifest and plugin files. Use a JSON linter or validator to identify and fix the syntax errors.

### MissingEnvironmentVariablesError
- **Reason**: Environment variables referenced in the manifest or plugin files are not set.
- **Solution**: Define all necessary environment variables in your environment or configuration.


# teamsApp/publishAppPackage
This action will publish built Teams app zip file to tenant app catalog.

## What the Action Does

This action performs the following steps:

1. Validates the input arguments.
2. Checks if the specified app package file exists.
3. Reads and extracts the manifest file from the app package.
4. Uses the Teams Dev Portal API to either publish a new app or update an existing one based on the app's presence in the Teams Admin Center.


## Syntax:
```
  - uses: teamsApp/publishAppPackage
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to this file. This is the path for built zip file.
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      publishedAppId: TEAMS_APP_PUBLISHED_APP_ID
```

## Input Validation Rules

The input arguments are defined within the `with` object. Below are the input parameters and their validation rules:

- **appPackagePath** (required): The path to the Teams app package that needs to be published.

### Example Input

Here is an example of how you can define the input parameters for this action:

```yaml
with:
  appPackagePath: "path/to/your/teams/app/package.zip"
```


## Output:
The output of the action execution is defined in the `writeToEnvironmentFile` object. The key represents the target output name, and the value is the environment variable name to store the output value.

- **publishedAppId**: The ID of the published Teams app.

### Example Output

Upon successful execution, the output will be set in the specified environment variable:

```yaml
writeToEnvironmentFile:
  publishedAppId: PUBLISHED_APP_ID
```
## Potential Errors for Troubleshooting

Understanding potential errors is crucial for troubleshooting issues. Below are the errors that may arise during the execution of this action:

1. **FileNotFoundError**
   - **Reason**: The specified app package file does not exist.
   - **Possible Solutions**: Ensure that the app package path is correct and that the file exists at the specified location.

2. **InvalidActionInputError**
   - **Reason**: One or more required input parameters are missing or invalid.
   - **Possible Solutions**: Validate that all required parameters are provided and correctly formatted.

3. **TeamsAppPublishConflictError**
   - **Reason**: A Teams app with the same external ID already exists in the staged app entitlements.
   - **Possible Solutions**: Consider using the update mechanism to modify the existing app or provide a different app package.

4. **TeamsAppPublishFailedError**
   - **Reason**: General failure during the publishing process to the Teams App Catalog.
   - **Possible Solutions**: Check the detailed error message for specific issues and ensure that all preconditions for publishing are met.

5. **UserCancelError**
   - **Reason**: The user cancels the publishing update operation.
   - **Possible Solutions**: Confirm the publishing update when prompted.

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

## Input Parameters

The `spfx/deploy` action requires certain inputs to function correctly, provided through the `with` object. The expected parameters and their validation rules are as follows:

### Required Parameters

- **`packageSolutionPath`** (`string`):
  - Description: The path to `package-solution.json` in the SPFx project.
  - This should be a valid path pointing to the `package-solution.json` file in the SPFx project.
  - Example:
    ```yaml
    with:
      packageSolutionPath: "config/package-solution.json"
    ```

### Optional Parameters

- **`createAppCatalogIfNotExist`** (`boolean`):
  - Description: Indicates whether to create a tenant app catalog if it does not already exist. The default value is `false`.
  - This should be a boolean value.
  - Example:
    ```yaml
    with:
      createAppCatalogIfNotExist: true
      packageSolutionPath: "config/package-solution.json"
    ```

### Complete Example
```yaml
uses: spfx/deploy
with:
  packageSolutionPath: "config/package-solution.json"
  createAppCatalogIfNotExist: true
```


## Potential Errors and Troubleshooting

During the execution of the `spfx/deploy` action, there are several potential errors that users may encounter. The corresponding error classes, reasons, and possible solutions are outlined below:

### 1. `GetSPOTokenFailedError`
- **Reason**: Failure to obtain the SharePoint Online (SPO) token.
- **Solution**: Ensure that your Microsoft 365 access token is valid and has the necessary permissions.

### 2. `CreateAppCatalogFailedError`
- **Reason**: Failure to create the app catalog in the tenant.
- **Solution**: Verify the tenant permissions and status. Make sure that the user has sufficient permissions to create an app catalog.

### 3. `NoValidAppCatelog`
- **Reason**: No valid app catalog found, and the `createAppCatalogIfNotExist` parameter is `false`.
- **Solution**: Set `createAppCatalogIfNotExist` to `true` or ensure that an app catalog exists in the tenant.

### 4. `FileNotFoundError`
- **Reason**: The specified `packageSolutionPath` does not exist.
- **Solution**: Ensure that the path to `package-solution.json` is correct and exists in the project directory.

### 5. `NoSPPackageError`
- **Reason**: The SPFx package file specified in `package-solution.json` is not found.
- **Solution**: Check the `package-solution.json` to ensure the correct path to the zipped package is specified and confirm the file exists.

### 6. `InsufficientPermissionError`
- **Reason**: Insufficient permissions to upload the app package to the app catalog site.
- **Solution**: Make sure that the user has the necessary permissions to upload apps to the app catalog.

### 7. `UploadAppPackageFailedError`
- **Reason**: Failure during the upload of the SPFx package.
- **Solution**: Investigate the underlying error message. This could be due to network issues or insufficient permissions.

### 8. `GetTenantFailedError`
- **Reason**: Failure to retrieve the tenant information.
- **Solution**: Ensure that the Microsoft 365 Graph API is accessible and that the user has the necessary permissions to query tenant information.

### 9. `GetGraphTokenFailedError`
- **Reason**: Failure to obtain the Microsoft Graph token.
- **Solution**: Verify that the Microsoft 365 token provider is configured correctly and has the required scopes.


# teamsApp/copyAppPackageToSPFx
This action will copy the Teams App zipped package to `teams` folder in SPFx directory to keep it updated. This is to ensure user will have aligned experience whether to publish Teams App from Teams Toolkit or manually sync to Teams in SharePoint app catalog.

## Functionality
- **Copy Teams App Package**: This action will copy the zipped Teams app package from the specified path to the `teams` directory within the specified SPFx project folder.
- **Icon Replacement**: It will also extract and replace custom icons (`color.png` and `outline.png`) in the SPFx project folder, if any are defined in the Teams app manifest.


## Syntax:
```
  - uses: teamsApp/copyAppPackageToSPFx
    with:
      appPackagePath: ${{TEAMS_APP_PACKAGE_PATH}}
      spfxFolder: ./src # Path to SPFx solution.
```

## Input Validation Rules
The action requires users to provide certain parameters within the `with` object:

- **`appPackagePath` (string)**: The path to the zipped Teams app package.
- **`spfxFolder` (string)**: The source folder of the SPFx project.

### Example Input
```yaml
uses: teamsApp/copyAppPackageToSPFx
with:
  appPackagePath: "./path/to/teamsapp.zip"
  spfxFolder: "./path/to/spfx/project"
```

### Validation
- Paths provided for `appPackagePath` and `spfxFolder` must be valid and accessible.
- The `appPackagePath` should exist and be readable.
  
## Potential Errors for Troubleshooting
Users of this action might encounter several types of errors during its execution. Below are some common errors, their possible reasons, and suggested solutions:

### Errors and Possible Solutions
1. **FileNotFoundError**
   - **Reason**: The specified `appPackagePath` does not exist.
   - **Solution**: Ensure that the `appPackagePath` is correct and the file exists at the location.

2. **ManifestNotFoundError**
   - **Reason**: The manifest file (`manifest.json`) is missing within the Teams app package.
   - **Solution**: Verify the Teams app package to ensure it includes a `manifest.json` file.

3. **PermissionDeniedError**
   - **Reason**: Insufficient permissions to read from the `appPackagePath` or write to the `spfxFolder`.
   - **Solution**: Ensure read and write permissions are granted for the specified paths.

4. **IconNotFoundError**
   - **Reason**: The specified icons (`color.png` or `outline.png`) are not found inside the Teams app package while their URLs are not external.
   - **Solution**: Check the Teams app package to verify that the icons are included and correctly referenced.

### Troubleshooting Steps
- **Path Verification**: Ensure that all provided paths (`appPackagePath` and `spfxFolder`) are correct and accessible.
- **Contents of Teams Package**: Verify the contents of the Teams app package to ensure it includes all necessary files (`manifest.json`, icons).
- **Access Rights**: Make sure the executing user has necessary permissions to read from and write to the specified paths.


# teamsApp/extendToM365
This action will upload your app as M365 title, so it can be viewed on Outlook and Office.

## Action Functionality
- **Upload Teams App Package:** The action uploads the provided Teams app package to a side-loading service.
- **Side-Load App:** It then facilitates side-loading into Microsoft 365.
- **Output Values:** After successful execution, the action outputs values including the M365 Title ID and App ID.


## Syntax
```yml
  - uses: teamsApp/extendToM365 # Extend your Teams app to Outlook and the Microsoft 365 app
    with:
      appPackagePath: ./appPackage/build/appPackage.${{TEAMSFX_ENV}}.zip # Required. Relative path to the built app package.
    writeToEnvironmentFile: # Write the information of created resources into environment file for the specified environment variable(s).
      titleId: M365_TITLE_ID # Required. The ID of M365 title.
      appId: M365_APP_ID # Required. The app ID of M365 title.
```

## Input Validation Rules
The action requires specific input parameters which must be defined within a `with` object. Below are the required inputs and their validation rules:

### Required Inputs
- **appPackagePath (string):** Path to the Teams app package. This should be a valid file path to the Teams app package ZIP file.
  - _Examples:_
    ```yaml
    with:
      appPackagePath: "./path/to/teams/app/package.zip"
    ```
## Output Specification
On successful execution, the action outputs specific values stored in environment variables defined within the `writeToEnvironmentFile` object. The keys represent the output names, and the values correspond to the environment variable names.

### Required Outputs
- **titleId:** The ID of the M365 title.
- **appId:** The App ID of the M365 title.

  - _Examples:_
    ```yaml
    writeToEnvironmentFile:
      titleId: "M365_TITLE_ID"
      appId: "M365_APP_ID"
    ```
## Potential Errors and Troubleshooting
When executing the `teamsApp/extendToM365` action, various errors can occur. Below is a list of potential errors, their causes, and possible solutions:

### InvalidActionInputError
- **Reason:** One or more inputs are invalid or missing.
- **Solution:** Ensure that the `appPackagePath` is correctly specified and is a string. Verify that all required environment variable names (titleId, appId) in `writeToEnvironmentFile` are specified.

### FileNotFoundError
- **Reason:** The file specified in `appPackagePath` does not exist.
- **Solution:** Check the file path provided and ensure that the file exists.

### UserError
- **Reason:** Issues such as invalid input data or access problems could trigger this error.
- **Solution:** Validate the input data, ensure proper permissions and tokens are available.

### SystemError
- **Reason:** System related issues like network problems or internal errors.
- **Solution:** Retry the action, checking for system logs and connectivity issues.

### PackageServiceError
- **Reason:** Related issues from the side-loading service, e.g., HTTP 400 due to invalid input or service misconfiguration.
- **Solution:** Verify the side-loading service endpoint and ensure the input package ZIP file is valid.


# file/createOrUpdateEnvironmentFile
## Overview

This action reads the specified environment file, updates or appends the given environment variables, and writes the updates back to the target environment file. It ensures that the environment file is created if it does not already exist.


## Syntax:
```yml
  - uses: file/createOrUpdateEnvironmentFile
    with: 
      target: /path/to/your/env/file # Required. This action will generate envs to the specified path.
      envs: 
        <your-env-key-1>: <your-env-value-1>
        <your-env-key-2>: <your-env-value-2>
```

## Inputs

The inputs for this action are specified within the `with` object. Below are the input parameters and the validation rules:

- `envs` (required): The environment variables to be created or updated in the environment file. The values can be of type string, boolean, or number.
- `target` (required): The path to the target environment file to be created or updated.

### Input Validation Rules

1. **`target`**:
   - Must be a non-empty string.

2. **`envs`**:
   - Must be an object.
   - Values must be either string, boolean, or number.

### Example Input

```yaml
uses: file/createOrUpdateEnvironmentFile
with:
  target: './.env'
  envs:
    NODE_ENV: 'production'
    DEBUG: false
    API_URL: 'https://api.example.com'
```
## Potential Errors for Troubleshooting

When executing the `file/createOrUpdateEnvironmentFile` action, you may encounter various errors. Below is a list of potential errors, reasons, and possible solutions:

### 1. InvalidActionInputError

**Reason:**
- The input parameters are incorrect.

**Possible Solution:**
- Ensure the `target` is a non-empty string.
- Ensure `envs` is an object and its values are of valid types (string, boolean, or number).

### 2. File System Error (Catch-all error)

**Reason:**
- Issues related to file access, creation, or writing.

**Possible Solution:**
- Ensure the file path is valid and accessible.
- Ensure sufficient permissions to read/write the target environment file.
- Verify that there is no conflict with existing file-lock mechanisms or permissions.

### 3. UserError or SystemError

**Reason:**
- Specific errors thrown by user code or system during the execution.

**Possible Solution:**
- Check the error logs for detailed error messages.
- Review the stack trace or error message details to isolate the specific issue.


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

## Input Validation Rules

The input for the `file/createOrUpdateJsonFile` action is defined within the "with" object. Below are the required and optional parameters along with their rules:

### Required Parameters

- `target`: 
  - **Description**: The target file path where the JSON content will be created or updated.
  - **Type**: String
  - **Validation**: Must be a non-empty string.

### Optional Parameters

- `appsettings`: 
  - **Description**: App settings to be generated.
  - **Type**: Object
  - **Validation**: If provided, must be of type object.

- `content`: 
  - **Description**: The JSON content to be created or updated, will be merged with existing content.
  - **Type**: Object
  - **Validation**: If provided, must be of type object.

### Example Input (YAML format)

```yaml
with:
  target: "./config/settings.json"
  content:
    logging: 
      level: "info"
      file: "log.txt"
  # Alternatively, you can use 'appsettings' instead of 'content':
  appsettings:
    featureFlags:
      enableNewFeature: true
```

**Note**: Only one of `content` or `appsettings` must be provided. If both are provided, the action will return an error.

## Potential Errors for Troubleshooting

This section enumerates potential errors that might be encountered during the execution of the action and provides possible solutions.

### Error Classes

- **UserError**: This error indicates incorrect input or configuration from the user.
- **SystemError**: This error signifies an unexpected issue within the system or environment where the script is running.

### Common Errors

1. **InvalidActionInputError**
   - **Reason**: This error occurs when one or more required parameters are missing or have invalid values.
   - **Solution**: Verify the input parameters. Ensure `target` is provided and is a valid string, and at least one of `content` or `appsettings` is provided as an object.

2. **FileNotExistError**
   - **Reason**: The specified target file does not exist, and there is an issue creating it.
   - **Solution**: Check the file path and permissions to ensure the file can be created at the specified location.

3. **JsonParsingError**
   - **Reason**: Errors encountered while reading or parsing the existing JSON file.
   - **Solution**: Ensure the existing file has valid JSON content and is accessible.

4. **FileWriteError**
   - **Reason**: Errors encountered while writing to the JSON file.
   - **Solution**: Check file system permissions to ensure the action has write access to the target file.

5. **UnexpectedError**
   - **Reason**: Any other unexpected error that is not handled specifically by the defined error classes.
   - **Solution**: Review the error message for more details and check logs for additional diagnostic information.


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
## Detailed Task Descriptions

### 1. Resolve Local Certificate

The `resolveLocalCertificate` function is responsible for setting up and optionally trusting a local SSL certificate.

#### Functional Steps:
1. **Generate Certificate**: Create new certificates if they do not exist at the specified paths.
2. **Trust Certificate**: Add the certificate to the store if not already trusted.
3. **Output Handling**: Set environment variables with paths to the generated certificate and key files.
4. **Logging**: Log and generate reports on the action's outcome.

#### Potential Errors and Troubleshooting

- **`SetupCertificateError`**:
  - **Reason**: Failure in setting up the SSL certificate.
  - **Solution**: Check certificate paths, permissions, and retry.

- **`TrustCertificateError`**:
  - **Reason**: Failure in trusting the SSL certificate.
  - **Solution**: Confirm permissions and system compatibility.

---

### 2. Install Azure Functions Core Tools

The `resolveFuncCoreTools` function manages the installation, versioning, and optional symlink creation of Azure Functions Core Tools.

#### Potential Errors and Troubleshooting

- **`FuncInstallationUserError`**:
  - **Reason**: Installation failure.
  - **Solution**: Verify system requirements and specified version.

- **`DepsCheckerError`**:
  - **Reason**: Dependency issues like missing Node.js.
  - **Solution**: Ensure compatible Node.js version is installed.

- **`LinuxNotSupportedError`**:
  - **Reason**: Unsupported OS.
  - **Solution**: Use supported OS like Windows or macOS.

- **Symlink Errors**:
  - **Reason**: Target directory for symlink already exists.
  - **Solution**: Clean up existing symlinks or provide a new directory.

---

### 3. Install .NET SDK

The `resolveDotnet` function determines if and how the .NET SDK is installed based on the input configurations.

#### Potential Errors and Troubleshooting

- **`DotnetInstallationUserError`**:
  - **Reason**: Installation failure.
  - **Solution**: Check system prerequisites and directory permissions.

- **`DepsCheckerError`**:
  - **Reason**: Dependency issues.
  - **Solution**: Investigate the error message for specific details.

- **`InvalidConfigurationError`**:
  - **Reason**: Misconfigured .NET SDK path.
  - **Solution**: Verify `dotnet.json` configuration.

- **`InstallationValidationFailed`**:
  - **Reason**: Installation validation failed.
  - **Solution**: Reattempt installation, confirming version specifications.

---

### 4. Install Teams App Test Tool

The `resolveTestTool` function manages the installation and required configuration for the Teams App Test Tool.

#### Potential Errors and Troubleshooting

- **`TestToolInstallationUserError`**:
  - **Reason**: Installation failure due to prerequisites or network issues.
  - **Solution**: Ensure Node.js installation and verify network connectivity.

- **`DepsCheckerError`**:
  - **Reason**: Version mismatch.
  - **Solution**: Confirm version range and adjust configuration.

- **`NodeNotFoundError`**:
  - **Reason**: Node.js not found.
  - **Solution**: Install Node.js before running the action.

- **General Dependency Errors**:
  - **Reason**: Issues like file not found or symlink creation failure.
  - **Solution**: Ensure correct permissions and review log messages for specifics.

---
## Other Troubleshooting guides

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

## Overview

The **script** action allows you to execute a user-defined script. This action is part of a predefined series of tasks in a CI/CD pipeline or workflow. It provides the flexibility to run shell commands or scripts and manage environment variables essential for your project.

### Functionality

When you execute the **script** action:

1. The specified script or command (`run`) is executed.
2. The action can run with a specified shell (`shell`) and within a defined working directory (`workingDirectory`).
3. Environment variables can be redirected and stored based on script output.

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
## Input Validation Rules

The inputs for the **script** action are defined in the `with` object. Here are the detailed validation rules:

### Required Properties

- **run**: _(string)_
  - Description: The command to run or the path to the script. The action is considered successful if the script exits with code 0.
  - Example: `"echo 'Hello World!'"`
  
### Optional Properties

- **workingDirectory**: _(string)_
  - Description: The directory to run the script in. Defaults to the directory of the configuration file.
  - Example: `"/path/to/directory"`

- **shell**: _(string)_
  - Description: The shell to use for executing the script. If not provided, defaults to the following based on the platform:
    - macOS: `/bin/zsh` if available, otherwise `/bin/bash`.
    - Windows: value of `ComSpec` environment variable if available, otherwise `cmd.exe`.
    - Linux/Other: `/bin/sh`.

- **timeout**: _(number)_
  - Description: Specifies a timeout for the script execution in milliseconds.
  - Example: `5000`

- **redirectTo**: _(string)_
  - Description: File path to redirect stdout and stderr.
  - Example: `"logs/output.log"`

### Example Input (YAML Format)

```yaml
with:
  run: "echo 'Hello World!'"
  workingDirectory: "/path/to/directory"
  shell: "/bin/bash"
  timeout: 5000
  redirectTo: "logs/output.log"
```

## Output:

All stdout start with "::set-teamsfx-env key=value" will be interpreted into outputs in .env file.

## Default shell command
If `shell` is not specified, use default shell. The rule is applied in the following order:
1. Use the value of the 'SHELL' environment variable if it is set. 
2. If current OS is macOS, then use '/bin/zsh' if it exists, otherwise use '/bin/bash'; 
3. If current OS is Windows, then use the value of the 'ComSpec' environment variable if it exists, otherwise use 'cmd.exe'; 
4. If current OS is Linux or other OS systems, use '/bin/sh' if it exists. 

## Potential Errors and Troubleshooting

Below are the potential errors that users might encounter when using the **script** action, along with their reasons and possible solutions.

### `ScriptTimeoutError`

- **Reason**: The script execution exceeded the specified timeout duration.
- **Solution**: Increase the `timeout` value in the input configuration to allow more time for the script to execute.

### `ScriptExecutionError`

- **Reason**: The script execution failed, possibly due to a non-zero exit code.
- **Solution**: Check the script for errors, ensure all commands are valid, and that all necessary dependencies are installed.

### `InvalidInputError`

- **Reason**: The input arguments provided do not match the required schema.
- **Solution**: Verify that all required properties are included in the `with` object and that the values match the expected type and format.

### `FileNotFoundError`

- **Reason**: The specified script or working directory does not exist.
- **Solution**: Ensure the `run` command points to a valid script and that the `workingDirectory` is correctly specified and exists.

### `ShellNotFoundError`

- **Reason**: The specified shell is not available on the machine.
- **Solution**: Use a valid shell path or remove the `shell` property to use the default shell for the platform.

### `EnvironmentVariableError`

- **Reason**: Errors reading environment variables for use within the script.
- **Solution**: Ensure all referenced environment variables are correctly defined and accessible.


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

# oauth/register

## Overview
The `oauth/register` action facilitates the creation of an OAuth registration within an application. It manages OAuth configurations, including creating a registration when it does not exist, and retrieving an existing registration if available. This action uses a source code implementation method `CreateOauthDriver.execute` to perform these operations.

## Input Validation Rules
The input parameters for the OAuth registration action are provided under the `with` object. Below are the parameters that must be fulfilled, including their data type, a description, and, where applicable, enumerated values:

- `name`:
  - **Description**: The name of the OAuth registration.
  - **Type**: `string`
  - **Validation**: Required, max length of 128 characters
- `appId`:
  - **Description**: The application ID for OAuth registration.
  - **Type**: `string`
  - **Validation**: Required
- `apiSpecPath`:
  - **Description**: Path to the API specification file.
  - **Type**: `string`
  - **Validation**: Required
- `flow`:
  - **Description**: Type of OAuth flow.
  - **Type**: `string`
  - **Validation**: Required, must be `"authorizationCode"`
- `applicableToApps` (Optional):
  - **Description**: Access scope of the OAuth registration.
  - **Type**: `string`
  - **Validation**: Must be one of `"SpecificApp"` or `"AnyApp"`, default is `"AnyApp"`
- `targetAudience` (Optional):
  - **Description**: Tenant access scope for the OAuth registration.
  - **Type**: `string`
  - **Validation**: Must be one of `"HomeTenant"` or `"AnyTenant"`, default is `"AnyTenant"`
- `clientId` (Optional):
  - **Description**: Client ID for OAuth registration.
  - **Type**: `string`
  - **Validation**: Required if present
- `clientSecret` (Optional):
  - **Description**: Client Secret for OAuth registration, required if `isPKCEEnabled` is `false`.
  - **Type**: `string`
  - **Validation**: Length should be within defined limits
- `refreshUrl` (Optional):
  - **Description**: The refresh URL for the OAuth registration.
  - **Type**: `string`
  - **Validation**: Should be a valid string URL
- `isPKCEEnabled` (Optional):
  - **Description**: Whether PKCE is enabled for OAuth registration.
  - **Type**: `boolean`
  - **Validation**: Defaults to `false`

### Example Input
```yaml
with:
  name: "MyOAuthRegistration"
  appId: "12345"
  apiSpecPath: "./path/to/api/spec"
  flow: "authorizationCode"
  applicableToApps: "AnyApp"
  targetAudience: "AnyTenant"
  clientId: "my-client-id"
  clientSecret: "my-client-secret"
  refreshUrl: "https://example.com/refresh"
  isPKCEEnabled: false
```

## Output Specification
The result of the action execution is written to environment variables specified in the `writeToEnvironmentFile` object. Below is the key-value mapping for the environment variables:

- `configurationId`:
  - **Description**: The configuration ID of the created OAuth registration.
  - **Environment Variable Name**: Defined by the user in the `writeToEnvironmentFile` object

### Example Output Configuration
```yaml
writeToEnvironmentFile:
  configurationId: "OAUTH_CONFIG_ID"
```

## Potential Errors for Troubleshooting
The following are potential errors that might occur during the execution of the OAuth registration action, along with their reasons and possible solutions:

- **OutputEnvironmentVariableUndefinedError**
  - **Reason**: The `outputEnvVarNames` parameter is undefined.
  - **Solution**: Ensure that `outputEnvVarNames` is correctly defined.

- **UserError or SystemError**
  - **Reason**: General errors that arise from user input or system issues.
  - **Solution**: Check the error message logs for specific details and resolve the indicated problems.

- **OauthNameTooLongError**
  - **Reason**: The provided registration name exceeds the maximum length of 128 characters.
  - **Solution**: Shorten the registration name.

- **InvalidActionInputError**
  - **Reason**: Invalid or missing parameters in the input arguments.
  - **Solution**: Verify that all required parameters are provided and valid according to the validation rules.

- **OauthDomainInvalidError**
  - **Reason**: The provided domain either exceeds the maximum allowed domains or is missing.
  - **Solution**: Check the domain configuration in the API specification and adjust it accordingly.

- **OauthFailedToGetDomainError**
  - **Reason**: Failed to retrieve the domain from the API specification.
  - **Solution**: Ensure the API specification path is correct and the domain configuration is available.

# oauth/update

The `oauth/update` action allows users to update an existing OAuth registration. This action involves validating the given input parameters, retrieving the current OAuth registration, comparing it to the input parameters, potentially asking for user confirmation, and then executing the update if necessary.

## Overview

The `UpdateOauthDriver.execute` function performs the following steps:

1. **Input Validation**: Validates the input arguments to ensure they are correctly provided.
2. **Retrieving Current Registration**: Retrieves the current OAuth registration using the provided `configurationId`.
3. **Comparison and Confirmation**: Compares the current registration with the provided inputs to determine if an update is necessary and, if so, asks the user for confirmation.
4. **Updating OAuth Registration**: If confirmed (or if confirmation is not required), updates the OAuth registration using the provided inputs.

## Input Validation Rules

The input arguments are provided in the `with` object. The following are the required inputs and their validation rules:

| Parameter         | Type      | Description                                                        | Required |
| ----------------- | --------- | ------------------------------------------------------------------ | -------- |
| `name`            | string    | The name of the OAuth registration.                               | Yes      |
| `appId`           | string    | The app ID of the OAuth registration.                             | Yes      |
| `apiSpecPath`     | string    | The path to the API specification file.                           | Yes      |
| `configurationId` | string    | The configuration ID of the OAuth registration.                   | Yes      |
| `applicableToApps`| string    | Which app can access the OAuth registration. Can be `SpecificApp` or `AnyApp`. Default is `AnyApp`.         | No       |
| `targetAudience`  | string    | Which tenant can access the OAuth registration. Can be `HomeTenant` or `AnyTenant`. Default is `AnyTenant`. | No       |
| `isPKCEEnabled`   | boolean   | Whether PKCE is enabled for the OAuth registration. Default is `False`.                                  | No       |

### Example Input (YAML format)

```yaml
uses: oauth/update
with:
  name: "MyOAuthRegistration"
  appId: "12345678-90ab-cdef-1234-567890abcdef"
  apiSpecPath: "./api-specs/oauth-spec.json"
  configurationId: "config-1234"
  applicableToApps: "SpecificApp"
  targetAudience: "HomeTenant"
  isPKCEEnabled: true
```

## Output Specification

Upon successful execution, the action writes the outputs to environment variables. The output is specified using the `writeToEnvironmentFile` object. The key represents the output name, and the value is the environment variable to store the output value.

Example:
```yaml
writeToEnvironmentFile:
  oauthUpdated: OAUTH_UPDATE_STATUS
```

The example above shows that the result of the OAuth update status is stored in the environment variable `OAUTH_UPDATE_STATUS`.

## Potential Errors for Troubleshooting

Here is a list of potential errors that could occur, along with their explanations and possible solutions:

1. **InvalidActionInputError**
   - **Reason**: One or more required input parameters are missing or invalid.
   - **Solution**: Ensure that all required parameters are correctly specified and meet the validation rules.

2. **OauthNameTooLongError**
   - **Reason**: The provided `name` exceeds the maximum length of 128 characters.
   - **Solution**: Ensure that the `name` parameter is within 128 characters.

3. **OauthDisablePKCEError**
   - **Reason**: Attempting to disable PKCE on an OAuth registration where it is currently enabled.
   - **Solution**: Reconsider whether you need to disable PKCE. If necessary, you may need to manually adjust the registration in the portal or adjust your workflow.

4. **OauthDomainInvalidError**
   - **Reason**: The domain list exceeds the maximum allowed domains for the OAuth registration.
   - **Solution**: Ensure that the domain list provided in the input is within the allowed limits.

5. **OauthFailedToGetDomainError**
   - **Reason**: Failed to retrieve the domain from the provided API specification.
   - **Solution**: Ensure that the `apiSpecPath` is correct and the specification file contains valid domain information.

6. **SystemError/UserError**
   - **Reason**: These are generic error classes. If the error is caught as a `UserError` or `SystemError`, it means something went wrong that needs further investigation.
   - **Solution**: Check the error message and logs for more details. Ensure all provided inputs and execution context are correct.


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