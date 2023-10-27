This is the guideline for resource plugin owners to contribute unit tests / end-to-end tests.

## Unit Test Guideline

Unit test path: *packages\fx-core\tests\plugins\resource\<myResroucePluginName>\unit\xxx.test.ts*

### Unit Test for Generating ARM Templates

Walk through the `plugin.generateArmTemplates()` API to check detailed logic that needs being tested. 

1. Should generate ARM templates successfully 
    
    - Arrange: Leverage testHelper’s method to mock plugin context. 
    - Act: Execute `plugin.generateArmTemplates()` API.
    - Assert: Utilizing `mockSolutionUpdateArmTemplates()` in *packages\fx-core\tests\plugins\resource\util.ts* to compile the return result. Verify that the ARM templates result compiled by solution mocked function is correct: expected properties are correct, undefined properties are undefined. 
 
1. Should generate ARM templates successfully in other scenarios. 

    Add test cases when the ARM templates generation logic differs when includes different cloud resources / capabilities. (E.g., The project is a SPFx project / tab project / bot project / tab+bot project / tab+bot+messagingExtension project, etc.)  

1. Should fail / throw error if xxx requirement is not met (e.g., some value does not exist in context)  

### Unit Test for Updating ARM templates 

1. Should update ARM templates successfully 

    - Arrange: Leverage testHelper’s method to mock plugin context. 
    - Act: Execute `plugin.updateArmTemplates()` API.
    - Assert: Utilizing `mockSolutionUpdateArmTemplates()` in *packages\fx-core\tests\plugins\resource\util.ts* to compile the return result. Verify that the ARM templates result compiled by solution mocked function is correct: expected properties are correct, undefined properties are undefined. 

1. Should update ARM templates successfully in other scenarios. 

    Add test cases when the ARM templates generation logic differs when includes different cloud resources / capabilities. (E.g., The project is a SPFx project / tab project / bot project / tab+bot project / tab+bot+messagingExtension project, etc.)  

1. Should fail / throw error if xxx requirement is not met (e.g., some value does not exist in context) 

## End-to-end Test Guideline

E2E test path: *packages\cli\tests\e2e\<myResroucePluginName>\xxx.tests.ts*. (E2E test file must end with *.tests.ts*.)

1. Create project with the plugin can generate ARM template / provision / deploy successfully
1. Azure resource can be deployed to customized resource group 
1. Add resource /capability should work successfully: Resource plugin bicep files generation will differ when project has different active plugins. Add E2E test to test the generated bicep files and the provision result when with different plugins in the project.

    For example, function resource plugin has the following template config bicep file:
    ```bicep
    // packages\fx-core\templates\plugins\resource\function\bicep\functionConfiguration.template.bicep
    {{#if (contains "fx-resource-key-vault" plugins)}}
    var m365ClientSecret = \{{fx-resource-key-vault.References.m365ClientSecretReference}}
    {{else}}
    var m365ClientSecret = provisionParameters['m365ClientSecret']
    {{/if}}

    resource appSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    name: '${functionAppName}/appsettings'
    properties: union({
        ...
        M365_CLIENT_SECRET: m365ClientSecret // Client secret of Microsoft Entra application
        ...
    }, currentAppSettings) // Merge new settings with existing settings
    }
    ```

    When with keyvault plugin, function resource plugin config bicep file will set the m365ClientSecret as keyvault reference and set into function app settings. So function plugin owner add an E2E test *TabAdFunctionKeyvault.tests.ts* to test that a tab + function + keyvault project can successfully scaffolded and provisioned. After provision, function app setting will have m365 client secret as a key vault reference.
