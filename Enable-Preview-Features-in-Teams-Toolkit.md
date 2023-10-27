Following content applies to VS Code Teams Toolkit v2.8.0+ and TeamsFx CLI v0.7.0+. You can find content for older tooling versions in page history.

> Please be advised these features are currently under active development, with a lot of changes taking place. Please expect breaking changes as we continue to iterate.
We really appreciate your feedback! If you encounter any issue or error, please report issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

# How to enable preview features
## VS Code Teams Toolkit
1. Upgrade to the latest [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension).
1. Open Visual Studio Code and find `Manage` icon from sidebar (Bottom Left) 
1. Select `Settings` and find `Teams Toolkit` under `Extensions` section.
1. Tick the checkbox for `Insider Preview` setting.
1. Restart Visual Studio Code.
> Note: If you enabled `Provision Azure Resources with ARM Template` preview feature, you need to tick the checkbox for `Insider Preview` again to continue use this preview feature.

## TeamsFx CLI
1. Upgrade to the latest [TeamsFx CLI](https://www.npmjs.com/package/@microsoft/teamsfx-cli).
1. Set `TEAMSFX_INSIDER_PREVIEW` environment variable to `true` (either globally or for current shell)

# Preview features

## Provision Azure Resources with ARM Template
Teams Toolkit provides seamless integration with Azure resources, and we integrate with ARM so that you can now declaratively provision Azure resources your application needs using infrastructure as code approach using [Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) as a domain-specific language  (DSL) to author [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) template.

> Please note following support is still in progress now, please stay tuned. 
> * Add APIM

If you want to customize the ARM template, just update the bicep files at `templates/azure` and Azure parameter files at `.fx/configs`, then trigger provision command again to provision with your customized ARM template.
The Azure parameter file supports referencing values from environment variable. Just put `{{YOUR_ENV_NAME}}` to the value part, and our tooling will replace it with actual value. There're also some predefined placeholders in the parameter file. We're working to provide more detailed documents for you. Please stay tuned.

> Knwown issues
> * When adding resource, the `.fx/azure.parameters.{env}.json` will also be regenerated, which will change the resource name being provisioned. Please go to `templates/azure/backup/{timestamp}/parameters/` folder and copy the value of `resourceBaseName` as well as other customized content to the new file to ensure the tooling functions well. We're working on features to avoid regenerating these files.

## Managing Multiple Environments in Teams Toolkit
 The Teams toolkit provide a simple way for developers to easily create multiple environments and deploy artifacts to a target environment for a Teams app.

### Create a New Environment Copy
By default, the toolkit generates: 
- a `local` environment to represent your local machine environment configurations. 
- a `dev` environment after creating a new project to represent a remote environment configurations.

To add another remote environment for the app, you can use the `Teams: Create new environment copy` command in the tree view:

![add-env](https://user-images.githubusercontent.com/10163840/137431115-3a68fde9-ee81-49a3-a100-c5cc13f51bc2.png)

And then input your new environment name:
![input-env](https://user-images.githubusercontent.com/10163840/137431272-253cb636-18cf-4292-991e-f61905d2c4cc.png)

If you have more than one existing environments, you will need to select an existing environment to create the environment copy. Behind the scenes, the file content of `config.<newEnv>.json` and `azure.parameters.<newEnv>.json` will be copied from the existing environment you selected.

### Customize Environment-related Configuration Files
The toolkit provides the flexibility to let users customize the resource provision behavior for an environment.

Here is some common scenarios we can support for customized provision:
| Scenarios | Where to customize |
|--|--|
| Azure Resource Customization | <ul> <li>bicep files under `templates/azure`.</li> <li>`.fx/azure.parameters.<envName>.json`.</li></ul> |
| App Manifest | <ul> <li>`templates/manifest.template.json`.</li> <li>`manifest` section in`.fx/config.<envName>.json`.</li>  </ul> |
| Reusing existing Microsoft Entra app for Teams app | <ul> <li>`auth` section in`.fx/config.<envName>.json`.</li> </ul> |
| Reusing existing Microsoft Entra app for bot | <ul> <li>`bot` section in`.fx/config.<envName>.json`.</li> </ul> |
| Skip add user when provision SQL | <ul> <li>`skipAddingSqlUser` property in`.fx/config.<envName>.json`.</li> </ul> |

- For how to use the environment config file, you can refer to the [environment configuration schema](https://aka.ms/teamsfx-config).
- For how to use BICEP files in Teams toolkit, you can refer to [official BICEP document](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/) for help.

### Select Target Environment on Provision/Deploy/Publish/Preview
As we introduce multi-environments concept in Teams toolkit, for all environment related operations, you will need to select a target environment to perform with. The toolkit will prompt and ask for a target environment when needed.

![select-env](https://user-images.githubusercontent.com/10163840/137431518-31c5fa2d-7867-441b-ab90-c5f87f78e8b9.png)

### Project's Folder Structure
Main changes in project's folder structure:
- `.fx/configs`: config files that user can customize for the Teams app.
    - `config.<envName>.json`: per-environment configuration file.
    - `azure.parameters.<envName>.json`: per-environment parameters file for Azure BICEP provision.
    - `projectSettings.json`: global project settings which apply to all environments.
    - `localSettings.json`: local debug configuration file.
- `.fx/publishProfiles`: provision result that generated by the Toolkit.
- `templates`
    - `appPackage`: app manifest template files.
    - `azure`: BICEP template files. 


Happy coding!

## Collaborating on TeamsFx Project
Previous version of Teams Toolkit is not easy for multiple users to develop the same project due to missing privilege to access Teams APP and Microsoft Entra APP. If multiple developers want to share remote resources and work together, they need to manually handle permissions of Teams App and Microsoft Entra APP which need deep understanding the low-level details about the TeamsFx project.

Teams Toolkit now natively support add other collaborators for TeamsFx project which is much easy and straightforward for collaborative development.

### Collaborating - Use VSCode

#### Usage
- To use collaboration feature, you need to **login M365 and Azure account** and your **TeamsFx project should be provisioned first**

- In the Teams Toolkit panel, click Environment, and expand an environment name, there will be a Collaborators node

  ![collaborator-node](https://user-images.githubusercontent.com/5545529/137440190-d247dcc2-efec-4050-b1ab-bbfd9ee8d0bf.png)


- Click grant permission button on the right side of Collaborators node, then you can add another M365 account email address as collaborator

  ![input-collaborator-email](https://user-images.githubusercontent.com/5545529/137440192-6e418e28-70fc-4130-b710-601104cdcffa.png)

- Now collaborator with the added account can develop, provision and deploy the project


#### E2E Work Flow in VSCode
##### As a Project Creator
- Open VSCode, create a new TeamsFx Tab project (You can also select bot), and the hosting type select Azure

- Login M365 account and Azure account

- Provision and deploy your project, after provision success, you will see your m365 account listing below Collaborators node in Teams Toolkit panel

    ![collaborator-node](https://user-images.githubusercontent.com/5545529/137440190-d247dcc2-efec-4050-b1ab-bbfd9ee8d0bf.png)

-	Add another account as collaborator from tree view by click the grant permission button. Note that the added account must under the same tenant
  
    ![add-collaborator](https://user-images.githubusercontent.com/5545529/137440187-06d31c94-0576-4fb0-bcc0-8fdb5f4490b3.png)

- Push your project to GitHub

##### As a Project Collaborator
- Clone the project from GitHub
-	Login M365 account use collaborator’s account
-	Login Azure account which has contributor permission for all the Azure resources
-	Update Tab code, and deploy the project to remote
-	Launch remote and the project should work fine


### Collaborating - Use CLI
Teams Toolkit CLI provides `teamsFx permission` Commands for collaboration scenario.

#### Commands
| `teamsFx permission` Commands | Descriptions |
|:------------------------------|-------------|
| `teamsfx permission grant` | Grant permission for collaborator's M365 account for the project |
| `teamsfx permission status` | Show permission status for the project | 

***

#### Parameters for `teamsfx permission grant`
- ##### `--env`
	**(Required)** Provide env name.

- ##### `--email`
	**(Required)** Provide collaborator's M365 email address. Note that the collaborators's account should be in the same tenant with creator.

#### Parameters for `teamsfx permission status`
- ##### `--env`
	**(Required)** Provide env name.

- ##### `--list-all-collaborators`
	With this flag, Teams Toolkit CLI will print out all collaborators for this project.

### Examples
Here are some examples for you to better handling permission for `TeamsFx` projects.

#### Grant Permission
```bash
teamsfx permission grant --env dev --email user-email@user-tenant.com
```

#### Show Permission Status
```bash
teamsfx permission status --env dev
```

#### List All Collaborators
```bash
teamsfx permission status --env dev --list-all-collaborators
```

### Limitations of collaboration feature
> Azure related permissions should be handled manually by the Azure subscription administrator via Azure portal, different Azure account should at least have contributor role for the subscription so that developers can work together to provision and deploy TeamsFx project
