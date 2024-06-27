## Prerequisites:

You should have a Teams bot app that has been deployed to Azure with the following resources:
1.	Azure bot service
2.	An Entra id that is used for bot authentication
3.	An Azure resource that hosts your bot app (app service, Azure functions, etc)

## Steps:

### 1.  Create a new Azure bot service with MSI type

Since Azure bot service’s id and type cannot be modified after creation, you need to create a new Azure bot service. Select type “User-Assigned Managed Identity” and creation type “Create new Microsoft App ID”, and it will create both the Azure bot service and the managed identity for you.

![image](https://github.com/OfficeDev/teams-toolkit/assets/25220706/4dc2073f-93f9-4d7b-9721-6903c7463056)

You can also manually create a managed identity first then create the Azure bot service with creation type “Use existing app registration”.

You need to update the new Azure bot service’s **messaging endpoint** and **Channels** to be the same as the old one.


### 2.  Add the managed identity to the resource that hosts your bot app

Go to your app’s hosting resource, select Settings->Identity->User assigned. Add the managed identity created in step 1.
 

### 3.  Update your code and deploy to your hosting resource

Update your bot’s config to use type “UserAssignedMsi”. Below is a typescript example:
```typescript
const credentialsFactory = new ConfigurationServiceClientCredentialFactory({
  MicrosoftAppType: 'UserAssignedMsi',
  MicrosoftAppId: '{your msi’s client id}',
  MicrosoftAppTenantId: '{your msi’s tenant id}',
});

const botFrameworkAuthentication = new ConfigurationBotFrameworkAuthentication(
  {},
  credentialsFactory
);

const adapter = new CloudAdapter(botFrameworkAuthentication);
```

The MicrosoftAppId’s value is your newly created managed identity’s client id. The MicrosoftAppTenantId’s value is your newly created managed identity’s tenant id.


### 4.  Test your bot app

### 5.  Clean up unneeded resources

If your bot works, you can delete the old Azure bot service and the old Entra id.
