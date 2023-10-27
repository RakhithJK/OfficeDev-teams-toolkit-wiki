# Architecture

Here is an overall architectural diagram for the `React with Fluent UI` app template:
![Architecture](https://github.com/OfficeDev/TeamsFx/assets/63089166/a73b8b48-1411-4b09-bd74-ffbc8974f675)

This app template has three major components:

1. Azure Storage: Host the React app with Fluent UI. The code of the React app can be found in the `src` folder.

2. Azure Function: Middle-tier service to call Graph. Adopted [On-Behalf-Of flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow). Code of the Azure Function can be found in `api` folder.

3. Microsoft Entra: Both `Azure Storage` and `Azure Function` are configured with Microsoft Entra app and use the Microsoft Entra app for authentication.

# Authentication
![Authentication](https://github.com/OfficeDev/TeamsFx/assets/63089166/49781751-0910-42d9-b218-9de2a7495ddb)

Some key points in the authentication flow:
1. Get SSO token and call Azure Function with SSO token

    This app uses `TeamsFx` SDK to get SSO token from Microsoft 365 apps. Related code can be found in `src\components\samples\AzureFunction.ts`.
2. Get access token with SSO token

    This app uses `createMicrosoftGraphClientWithCredential` to create a graph client and will automatically request for access token with provided SSO token. Related code can be found in `api\index.ts`.