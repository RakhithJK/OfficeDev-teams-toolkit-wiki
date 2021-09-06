> Please be advised these features are currently under active development, with a lot of changes taking place. Please expect breaking changes as we continue to iterate.
We really appreciate your feedback! If you encounter any issue or error, please report issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

# Provision Azure Resources with ARM Template
Teams Toolkit provides seamless integration with Azure resources, and we integrate with ARM so that you can now declaratively provision Azure resources your application needs using infrastructure as code approach using [Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) as a domain-specific language  (DSL) to author [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) template.

> Please note following support is still in progress now, please stay tuned. 
> * Add APIM
> * Provision existing project using ARM template
> * Create a new project from sample gallery and provision using ARM template

## How to enable this feature
1. Upgrade to the latest [Teams Toolkit](https://github.com/OfficeDev/TeamsFx/releases/download/ms-teams-vscode-extension%402.6.0-rc.0/ms-teams-vscode-extension-2.6.0-rc.0.vsix).
1. Open Visual Studio Code and find `Manage` icon from sidebar (Bottom Left) 
1. Select `Settings` and find `Teams Toolkit` under `Extensions` section.
1. Tick the checkbox for `Arm Support Enabled` and `Validate Bicep` settings.
1. Restart Visual Studio Code.

Now you can create a new Teams application and provision it with ARM. Happy coding!