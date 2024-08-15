# Worked with shared access signatures (SAS) disabled with bot added
First of all, all those changes are need running as the user that granted to be [Contributor](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/general#contributor), [Owner](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/general#owner) or [Role Based Access Control Administrator](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/general#role-based-access-control-administrator). Otherwise, the `Microsoft.Authorization/roleAssignments` part in the Bicep will running fail.

## Bicep change
### Add SAS setting to make sure the SAS is disabled
```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageName
  kind: 'StorageV2'
  location: location
  sku: {
    name: storageSKU
  }
  properties: {
    allowSharedKeyAccess: false // this line makes the SAS disabled.
  }
}

```

### reuse the bot managed identity
Use you bot's identity to authorized between Azure Function App/App Service and Azure Storage.
```bicep
// The managed identity that use to connect between bot and Azure Function
resource identity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' = {
  location: location
  name: identityName
}

// Grant Blob Data Contributor role to this managed identity
// https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles
var StorageBlobDataContributorRole = 'ba92f5b4-2d11-453d-a403-e96b0029c9fe'

resource storageRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid('resourceBaseName-${uniqueString('StorageRoleAssignment')}')
  scope: storage
  properties: {
    principalId: identity.properties.principalId
    principalType: 'ServicePrincipal'
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', StorageBlobDataContributorRole)
  }
}

resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  kind: 'functionapp'
  location: location
  name: functionAppName
  properties: {
    serverFarmId: serverfarm.id
    httpsOnly: true
    siteConfig: {
      alwaysOn: true
      appSettings: [
        // add the storage's name to the environment variable
        {
          name: 'STORAGE_NAME'
          value: storageName
        }
        // managed identity id should be also add to the environment varibale
        {
          name: 'MANAGED_IDENTITY_ID'
          value: identity.properties.clientId
        }
      ]
      ftpsState: 'FtpsOnly'
    }
  }
  // function app also need this managed identity to connect with bot
  identity: {
    type: 'UserAssigned'
    userAssignedIdentities: {
      '${identity.id}': {}
    }
  }
}
```

## Code changes
Code change is based on this [docs](https://learn.microsoft.com/en-us/entra/identity-platform/multi-service-web-app-access-storage?tabs=azure-portal%2Cprogramming-language-nodejs#grant-access-to-the-storage-account)
```javascript
const credential = new DefaultAzureCredential({ managedIdentityClientId: process.env["MANAGED_IDENTITY_ID"] });
const blobServiceClient = new BlobServiceClient(`https://${process.env["STORAGE_NAME"]}.blob.core.windows.net`, credential);
const containerClient = blobServiceClient.getContainerClient("YOUR_BLOB_CONTAINER_NAME");

// test read
const contentBuffer = await containerClient.getBlockBlobClient("YOUR_FILE_NAME").downloadToBuffer();
console.log(content.toString());

// test write
const test = Buffer.from("TEST CONTENT");
const write  = containerClient.getBlockBlobClient("TEST_WRITE_FILE_NAME");
await write.upload(test, test.length);
```

# Worked with shared access signatures (SAS) disabled without bot
## Bicep change
Use managed identity to connect between Function/App Service and Azure Storage
```bicep
resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  kind: 'functionapp'
  location: location
  name: functionAppName
  properties: {
    serverFarmId: serverfarm.id
    httpsOnly: true
    siteConfig: {
      alwaysOn: true
      appSettings: [
        // add the storage's name to the environment variable
        {
          name: 'STORAGE_NAME'
          value: storageName
        }
      ]
      ftpsState: 'FtpsOnly'
    }
  }
  // use the system assigned identity in function or app service
  identity: {
    type: 'SystemAssigned'
  }
}

resource storage 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageName
  kind: 'StorageV2'
  location: location
  sku: {
    name: storageSKU
  }
  properties: {
    // disable SAS
    allowSharedKeyAccess: false
  }
}

// Grant Blob Data Contributor role to this managed identity
// https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles
var StorageBlobDataContributorRole = 'ba92f5b4-2d11-453d-a403-e96b0029c9fe'

resource storageRoleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid('resourceBaseName-${uniqueString('StorageRoleAssignment')}')
  scope: storage
  properties: {
    principalId: functionApp.identity.principalId
    principalType: 'ServicePrincipal'
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', StorageBlobDataContributorRole)
  }
}

```

## Code change
In this solution, we use system assigned identity, so we just use the `DefaultAzureCredential` to get our identity.
```javascript
const credential = new DefaultAzureCredential();
const blobServiceClient = new BlobServiceClient(`https://${process.env["STORAGE_NAME"]}.blob.core.windows.net`, credential);
const containerClient = blobServiceClient.getContainerClient("YOUR_BLOB_CONTAINER_NAME");

// test read
const contentBuffer = await containerClient.getBlockBlobClient("YOUR_FILE_NAME").downloadToBuffer();
console.log(content.toString());

// test write
const test = Buffer.from("TEST CONTENT");
const write  = containerClient.getBlockBlobClient("TEST_WRITE_FILE_NAME");
await write.upload(test, test.length);
```

# Worked with ACL enabled (Disable public access)
When public access disabled, we need create a Vnet to connect between Azure Function/App Service to Azure Storage.
This change will only be implemented by the Bicep.

```bicep
// Define a Network Security Group
resource networkSecurityGroup 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: '${resourceBaseName}-NSG'
  location: location
  properties: {
    securityRules: []
  }
}

// Define a Virtual Network
resource vnet 'Microsoft.Network/virtualNetworks@2023-11-01' = {
  name: '${resourceBaseName}-VNet'
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.0.0.0/16']
    }
    subnets: [
      {
        name: 'default'
        properties: {
          addressPrefix: '10.0.10.0/24'
          networkSecurityGroup: {
            id: networkSecurityGroup.id
          }
          serviceEndpoints: [
            {
              service: 'Microsoft.Storage'
              locations: ['*']
            }
          ]
          delegations: [
            {
              name: 'delegation'
              properties: {
                serviceName: 'Microsoft.Web/serverfarms'
              }
              type: 'Microsoft.Network/virtualNetworks/subnets/delegations'
            }
          ]
          privateEndpointNetworkPolicies: 'Disabled'
          privateLinkServiceNetworkPolicies: 'Disabled'
        }
      }
    ]
  }
}

resource storage 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageName
  kind: 'StorageV2'
  location: location
  sku: {
    name: storageSKU
  }
  properties: {
    allowBlobPublicAccess: false
    networkAcls: {
      bypass: 'AzureServices'
      // deny public access
      defaultAction: 'Deny'
      // add to the vnet
      virtualNetworkRules: [
        {
          id: vnet.properties.subnets[0].id
          action: 'Allow'
        }
      ]
    }
  }
}

resource functionApp 'Microsoft.Web/sites@2021-02-01' = {
  kind: 'functionapp'
  location: location
  name: functionAppName
  properties: {
    serverFarmId: serverfarm.id
    httpsOnly: true
    // just add this line to add function to the vnet
    virtualNetworkSubnetId: vnet.properties.subnets[0].id
  }
}
```

# Reference
* [Access Azure Storage from a web app using managed identities](https://learn.microsoft.com/en-us/entra/identity-platform/multi-service-web-app-access-storage?tabs=azure-portal%2Cprogramming-language-csharp#grant-access-to-the-storage-account)