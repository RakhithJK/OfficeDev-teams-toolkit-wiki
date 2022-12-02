Azure SQL Database is an always-up-to-date, fully managed relational database service built for the cloud. You can easily build applications with Azure SQL Database and continue to use the tools, languages, and resources you're familiar with.

# Steps to create Azure SQL Database

Teams Toolkit orchestrates cloud service provision and configuration with an infrastructure as code approach using a Domain Specific Language called [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep).

You can follow these steps to add Azure SQL Database to your app with bicep:

## Step 1: Add Azure SQL Database declaration to your bicep file

After you created a project using Teams Toolkit, the bicep file is usually located at `infra/azure.bicep`. Open this file and append following content to it. If your project is not created using Teams Toolkit, append the content to your own bicep file.

```
@description('The name of the SQL logical server.')
param serverName string = uniqueString('sql', resourceGroup().id)

@description('The name of the SQL Database.')
param sqlDBName string = 'SampleDB'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('The administrator username of the SQL logical server.')
param administratorLogin string

@description('The administrator password of the SQL logical server.')
@secure()
param administratorLoginPassword string

resource sqlServer 'Microsoft.Sql/servers@2021-08-01-preview' = {
  name: serverName
  location: location
  properties: {
    administratorLogin: administratorLogin
    administratorLoginPassword: administratorLoginPassword
  }
}

resource sqlDB 'Microsoft.Sql/servers/databases@2021-08-01-preview' = {
  parent: sqlServer
  name: sqlDBName
  location: location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}

// Allow Azure services connect to the SQL Server
resource sqlFirewallRules 'Microsoft.Sql/servers/firewallRules@2021-08-01-preview'' = {
  parent: sqlServer
  name: 'AllowAzure'
  properties: {
    endIpAddress: '0.0.0.0'
    startIpAddress: '0.0.0.0'
  }
}

```

> Note: above content generates a server name based on your resource group name and sets database name to `SampleDB` by default. If you want to change the names, you can set additional parameters `serverName` and `sqlDBName` in step 2.

## Step 2: Add parameters for Azure SQL Database bicep snippet

We need to add some required parameters for the bicep snippet in step 1.
1. Add following parameter to bicep's parameter file and set the value of `administratorLogin` to desired login name. For projects created using Teams Toolkit, the parameter file usually located at `infra/azure.parameters.json`.
   ```
    "administratorLogin": {
      "value": ""
    },
    "administratorLoginPassword": {
      "value": "${{SQL_ADMIN_PASSWORD}}"
    }
   ```
2. Add following content to `teamsfx/.env.{env_name}` and set the value of `SQL_ADMIN_PASSWORD`
   ```
   SQL_ADMIN_PASSWORD=
   ```
> Note: ${{ENV_NAME}} is a special placeholder supported by Teams Toolkit, which references the value of an environment variable. You can replace the real values in `azure.parameters.json` with this placeholder and set the environment variable values in `teamsfx/.env.{env_name}` or set to your machine's environment variable directly.

## Step 2: connect your computing resource to Azure SQL Database

There are 2 ways to connect to your Azure SQL Database in Azure: using username/password and using Azure Managed Identity.

### Connect using username/password

To connect to Azure SQL Database using the traditional username/password way, you can compose the connection string based on your programming language and library, then configure the connection string to your computing resource. For example, you can configure your connection string to Azure App Service using bicep as below:
```
resource webApp 'Microsoft.Web/sites@2021-02-01' = {
  kind: 'app'
  location: location
  name: webAppName
  properties: {
    serverFarmId: serverfarm.id
    siteConfig: {
      appSettings: [
        // other app settings...
        {
          name: 'your_app_setting_name' // the name to reference your SQL connection string
          value: 'Data Source=${sqlServer.properties.fullyQualifiedDomainName};Initial Catalog=${sqlDBName};User ID=${administratorLogin};Password=${administratorLoginPassword}' // sample connection string for C#, this is only used for demonstration purpose. DO NOT use admin credential to connect your SQL databases
        }
      ]
    // other site configs...
    }
  }
}
```

### Connect using Azure Managed Identity

Managed identities provide an automatically managed identity in Azure Active Directory for applications to use when connecting to resources that support Azure Active Directory (Azure AD) authentication.

You can refer this document to understand how to connect to Azure SQL Database using Managed Identity: https://learn.microsoft.com/en-us/azure/app-service/tutorial-connect-msi-azure-database

## Step 3: Update your cloud infrastructure

After you updated bicep file for your project, you need to run `Teams: Provision in the cloud` command in VS Code extension to apply your changes.



