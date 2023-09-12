# Overview

Microsoft Teams supports the ability to run web-based UI inside "custom tabs" that users can install either for just themselves (personal tabs) or within a team or group chat context.

SSO Enabled Tab via APIM Proxy shows you how to build a single-page Tab app with Graph Toolkit in frontend, Azure API Management (APIM) as Proxy for calling Graph APIs. APIM has adopted [On-Behalf-Of flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) for SSO.

With this sample, you can achieve the SSO feature in your tab app using OBO (on-behalf-of) flow without building a dedicated backend service.

## This sample illustrates

- How to use Teams Toolkit to create a Teams tab app.
- How to use integrate APIM in TeamsFx projects.
- How to implement SSO in Teams Tab app.
- How to use APIM as proxy of Graph Toolkit, use SSO token to call Graph and get user login info.

## Note
- This sample has adopted [On-Behalf-Of Flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to implement SSO.

- This sample uses Azure API Management as proxy, and make authenticated requests to call Graph.

- Due to system webview limitations, users in the tenant with conditional access policies applied cannot consent permissions when conduct an OAuth flow within the Teams mobile clients, it would show error: "xxx requires you to secure this device...".

## Benefits

1. This sample is an single page application, which can avoid login issue on mobile platform by implementing OBO Flow.

1. Secure your app by only using Access Token inside API Management.

1. Easily integrate with Graph Toolkit to access Graph in the Teams app.

# Architecture

Here is an overall architectural diagram for the `SSO Enabled Tab via APIM Proxy` sample:

![Architecture](https://github.com/OfficeDev/TeamsFx-Samples/assets/63089166/a256f1ab-1b23-4264-9f0d-ed8ff45aea09)


# Authentication

![Authentication](https://github.com/OfficeDev/TeamsFx-Samples/assets/63089166/fe7ceb81-fe2f-4d12-9d87-8191edb9ba19)

# Minimal path to awesome

## Run this app locally

1. Hit `F5` to start debugging. Alternatively open the `Run and Debug Activity` Panel and select `Debug (Edge)` or `Debug (Chrome)`.

    > APIM is required in this sample. TeamsFx will provision APIM on Azure, and will ask you to input `subscription` and `resource group name`.

## Run this app on Azure
1. Click `Provision` from `LIFECYCLE` section or open the command palette and select: `Teams: Provision`.

1. Click `Deploy` or open the command palette and select: `Teams: Deploy`.

1. Open the `Run and Debug Activity` Panel. Select `Launch Remote (Edge)` or `Launch Remote (Chrome)` from the launch configuration drop-down.