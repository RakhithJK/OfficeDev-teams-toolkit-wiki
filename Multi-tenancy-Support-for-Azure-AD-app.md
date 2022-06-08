## How to support Multi-tenancy in TeamsFx project

When SSO is enabled, Teams Toolkit will by default provision a [single-tenant](https://docs.microsoft.com/azure/active-directory/develop/single-and-multi-tenant-apps#who-can-sign-in-to-your-app) Azure AD app, which means only user and guest accounts in thea same directory as your M365 account can successfully sign in to your Teams app. 

To support [multi-tenant](https://docs.microsoft.com/azure/active-directory/develop/single-and-multi-tenant-apps#who-can-sign-in-to-your-app), you can follow the steps below to update your TeamsFx project.

> Note: This document is only for TeamsFx projects that have already enabled [single sign on](https://aka.ms/teamsfx-add-sso).

### (Optional) Update your Tab applications

> You can skip this part if your projects does not contain Tab application.

> Since Azure AD app requires an ["tenant verified domain"](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-modify-supported-accounts#why-changing-to-multi-tenant-can-fail) for `Application ID URI`, you can use your own Custom Domain or Create a new Custom Domain on Azure.

1. [Provision](https://docs.microsoft.com/microsoftteams/platform/toolkit/provision) your TeamsFx project.

1. Follow steps in [Action 1](https://github.com/OfficeDev/TeamsFx/blob/main/docs/fx-core/aad-help.md#action-1-note-frontend-info) and note your Frontend Domain.

1. (Optional) Follow steps in [Action 2](https://github.com/OfficeDev/TeamsFx/blob/main/docs/fx-core/aad-help.md#action-2-provision-cdn-profile-on-azure-portal) to provision CDN Profile on Azure Portal

    > Note: If you have a Custom Domain, you can skip this part. Remember to point your Custom Domain to the Frontend Domain noted in step 2.

1. Follow steps in [Action 3](https://github.com/OfficeDev/TeamsFx/blob/main/docs/fx-core/aad-help.md#action-3-update-frontend-info) to update the frontend info in your project.

    > Note: you can skip the last `Provision` and `Deploy` step since we will do this after everything is setup.

### Update your project

1. Open `templates/appPackage/aad.manifest.json`, find `signInAudience` and set value as `AzureADMultipleOrgs`.

1. Open `.fx/configs/azure.parameter.${env}.json` and find the following line:
    ```
    "m365TenantId": "{{state.fx-resource-aad-app-for-teams.tenantId}}",
    ```
    and replace with:
    ```
    "m365TenantId": "common",
    ```

### Provision and Deploy your project

Run [Provision](https://docs.microsoft.com/microsoftteams/platform/toolkit/provision) and [Deploy](https://docs.microsoft.com/en-us/microsoftteams/platform/toolkit/deploy) in your project.