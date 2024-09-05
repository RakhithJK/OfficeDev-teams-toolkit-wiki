## How to support Multi-tenancy single sign-on in TeamsFx project

When SSO is enabled, Teams Toolkit will by default provision a [single-tenant](https://docs.microsoft.com/azure/active-directory/develop/single-and-multi-tenant-apps#who-can-sign-in-to-your-app) Microsoft Entra app, which means only user and guest accounts in the same directory as your M365 account can successfully sign in to your Teams app. 

To support [multi-tenant](https://docs.microsoft.com/azure/active-directory/develop/single-and-multi-tenant-apps#who-can-sign-in-to-your-app), you can follow the steps below to update your TeamsFx project.

> Note: This document is only for TeamsFx projects that have already enabled [single sign on](https://aka.ms/teamsfx-add-sso).

### (Optional) Update Application Id URI
> This part is only for TAB projects for ts/js. If you are working on a Bot/Messaging Extension or a VS project, please go to [Update your project](#update-your-project).

> Since Microsoft Entra app requires an ["tenant verified domain"](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-modify-supported-accounts#why-changing-to-multi-tenant-can-fail) for `Application ID URI`, you can use your own Custom Domain or Create a new Custom Domain on Azure.

1. [Provision](https://docs.microsoft.com/microsoftteams/platform/toolkit/provision) your TeamsFx project.

2. Note Frontend Info.

  - Open `.fx\env\.env.{envName}` file
  - Note the `TAB_ENDPOINT` and find the resource group.

#### Action 2 Provision CDN Profile on Azure Portal
1. Login to Azure portal, create a CDN profile and a CDN endpoint, select endpoint type as Storage static website, then point to your frontend hosting storage. [Learn More](https://docs.microsoft.com/en-us/azure/cdn/cdn-create-new-endpoint)

    ![image](https://github.com/user-attachments/assets/7547913a-447d-41df-84c6-1d81556ecbe6)

1. Navigate to your created CDN endpoint and copy the endpoint hostname. For example, "https://sample.azureedge.net"

3. (Optional) Provision CDN Profile on Azure Portal
  - Login to Azure portal, create a CDN profile and a CDN endpoint, select endpoint type as Storage static website, then point to your frontend hosting storage. [Learn More](https://docs.microsoft.com/en-us/azure/cdn/cdn-create-new-endpoint)

    ![image](../images/fx-core/aad/appIdUri-cdn-portal.png)

  - Navigate to your created CDN endpoint and copy the endpoint hostname. For example, "https://sample.azureedge.net"

  > Note: If you have a Custom Domain, you can skip this part. Remember to point your Custom Domain to the Frontend Domain noted in step 2.

4. Update Frontend Info.

  - Open `infra\azure.bicep` file, and find the following two lines:
    ```
    output TAB_DOMAIN string = siteDomain
    output TAB_HOSTNAME string = siteDomain
    output TAB_ENDPOINT string = 'https://${siteDomain}'
    ```

  - Replace `siteDomain` with your CDN endpoint as following. Note you need to use your CDN endpoint copied above.
   ```
    output TAB_DOMAIN string = 'sample.azureedge.net'
    output TAB_HOSTNAME string = 'sample.azureedge.net'
    output TAB_ENDPOINT string = 'https://sample.azureedge.net'
   ```

  - Run "Teams - Provision in the cloud" and "Teams - Deploy to the cloud" or press F5 to start local debug.

    Please refer to the [Setup CDN as storage custom domain](#scenario-one-setup-cdn-as-storage-custom-domain) to config custom domain.

  > Note: you can skip the last `Provision` and `Deploy` step since we will do this after everything is setup.

### (Optional) Update Azure Function
> This part is only for TAB projects with Azure Function.

1. Open `./infra/azure.bicep`, find the following lines:
    ```
    resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
        ...
        properties: {
            ...
            cors: {
                allowedOrigins: [ tabDomain ]
            }
            ...
        }
        ...
    ```
    and replace the tabDomain in cors.allowedOrigins with your Custom Domain.

### Update your project

1. Open `./aad.manifest.json`, find `signInAudience` and set value as `AzureADMultipleOrgs`.

1. Open `infra/azure.parameter.${env}.json` and find the following line:
    ```
    "m365TenantId": "{{state.fx-resource-aad-app-for-teams.tenantId}}",
    ```
    and replace with:
    ```
    "m365TenantId": "common",
    ```

### Provision and Deploy your project

Run [Provision](https://docs.microsoft.com/microsoftteams/platform/toolkit/provision) and [Deploy](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/deploy) in your project.