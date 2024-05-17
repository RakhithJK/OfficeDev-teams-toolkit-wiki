1. [Node.js and NPM compatiblity issues](#compatibility)
2. [Failure to install prerequisites](#prerequisites)
3. [Failure to add additional SPFx tab](#addfeature)
4. [Failed to scaffold](#scaffold)
5. [Failed to import existing SPFx solution](#import)
6. [Failed to deploy](#deploy)
7. [Add web part using Yeoman SharePoint generator of mismatched version](#addWebPart)

_Note:  #5 - #7 applies to TTK v5.x with local/global SPFx package selection while #1 - #4 applies to TTK v4.x before the feature deliver._

## 1. Node.js and NPM compatiblity issues<a name="compatibility"></a>
The following table lists SharePoint Framework and compatible versions of common tools and libraries, according to the [SharePoint Framework compatibility page](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/compatibility#spfx-development-environment-compatibility):
|SPFx|Node.js|NPM|TypeScript|React|
|--------------|-----------|------------|-----------|------------|
| 1.14 | LTS v12, LTS v14 | v5, v6 | v3.9 | v16.13.1 |
| 1.15 | LTS v14, LTS v16 | v6, v7, v8 | v4.5 | v16.13.1 |
| 1.16 | LTS v16.13+ | v7, v8 | v4.5 | v17.0.1 |

### Error message
Teams Toolkit automatically checks Node.js and NPM versions for the latest SharePoint Framework it supports (SPFx v1.16.1 as of writing). You will encounter the following errors during scaffolding if Teams Toolkit detects unsupported Node.js or NPM versions:

#### SPFx.NodeVersionNotSupported

![spfx-compat-check-node](https://github.com/OfficeDev/TeamsFx/assets/73154171/81c1c420-849f-465b-bf87-7dac9782f99b)

#### SPFx.NpmVersionNotSupported

![spfx-compat-check-npm](https://github.com/OfficeDev/TeamsFx/assets/73154171/3258a205-6605-4912-a1a1-d57314c5fdef)


#### SPFx.NpmNotFound

Teams Toolkit also checks if NPM is installed.
![spfx-install-check-npm](https://github.com/OfficeDev/TeamsFx/assets/73154171/8da3060a-70ef-4243-911b-67920100b62d)


### Remediation

Check your npm and Node.js version. The SharePoint Framework [v1.16.1](https://docs.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment#install-nodejs) is supported on the following Node.js versions:

- Node.js v16 LTS (v16.13.x - v16.18.x, aka: Gallium)

**Corresponding npm version is v7, v8 for SPFx v1.16**. Please make sure you have the right version installed for both npm and Node.js.

## 2. Failure to install prerequisites<a name="prerequisites"></a>

For SPFx app, Teams Toolkit uses Yeoman Generator for scaffolding. This requires both [Yeoman CLI](https://github.com/yeoman/yo) and the correct SPFx generator version to be installed.

### Error message
SPFx scaffolding could fail due to unsuccessful installation of above prerequisites:

#### _"We've encountered some issues when trying to install prerequisites under HOME/.fx folder"_

### Remediation

As the default behavior, Teams Toolkit will try to install them locally under `HOME/.fx`. Should the installation fail, we would revert to use your globally installed ones.

#### Step 1: Disable Prerequisite Checker

Go to _Manage > Settings > Extension > Teams Toolkit > SPFx Prerequisite Check_ or run 'Preferences: Open User Settings'.

![setting](https://github.com/OfficeDev/TeamsFx/assets/73154171/2d8b4943-ed20-47d8-bed6-efa8537181d9)

And uncheck these 2:

- Ensure Yeoman CLI is installed
- Ensure SPFx generator is installed

#### Step 2: Manually install or upgrade

In the output message in VSC, you should see the versions for Yeoman CLI and SPFx generator that Teams Toolkit supports. In this example output message, you can see that they are `4.3.0` and `1.14.0`:
![output](https://github.com/OfficeDev/TeamsFx/assets/73154171/627b5a2f-fafb-4d93-bd84-33a139e35945)


In the following, navigate to **your applicable scenario**:

##### 1. If you have Yeoman CLI and SPFx generator installed with the correct versions

Teams Toolkit will use them for scaffolding, there's no further action that needs to be taken now. You can now retry creating a new SPFx Teams app.

##### 2. If no Yeoman CLI is installed in your system

1. Run this any place in a terminal:

```sh
npm install --global yo
```

2. Install the SPFx generator version that Teams Toolkit supports, say `1.14`:

```sh
npm install @microsoft/generator-sharepoint@1.14 -g
```

##### 3. If you have Yeoman CLI installed but it's not the correct version

Install the Yeoman CLI version that Teams Toolkit supports, say `4.3.0`:

```sh
npm install --global yo@4.3.0
```

##### 4. If you have Yeoman CLI installed but no SPFx generator

Install the SPFx generator version that Teams Toolkit supports, say `1.14`:

```sh
npm install @microsoft/generator-sharepoint@1.14 -g
```

##### 5. If you have SPFx generator installed but it's not the correct version

1. If the global version is higher than the supported version

You can continue with your currently installed version but please note that some of the latest features might not be supported in Teams Toolkit.

2. If the global version is lower than supported
Install the SPFx generator version that Teams Toolkit supports, say `1.14`:

```sh
npm install @microsoft/generator-sharepoint@1.14 -g
```

## 3. Failure to add additional SPFx tab<a name="addfeature"></a>

Multi-tab SPFx project is supported in Teams Toolkit. To add an additional SPFx tab, you can try "Add features" and select "SPFx tab". Behind the scenes, Yeoman Generator is executed according to the configuration file (.yo-rc.json) in the current solution.

If .yo-rc.json file doesn't exist in your SPFx project, adding SPFx tab will fail with the following error:

#### SPFx.NoConfigurationFile

![spfx-no-configuration-file](https://github.com/OfficeDev/TeamsFx/assets/73154171/f8687e61-e3e4-470f-bd0c-6817911aaee7)

### Remediation

Please follow the instructions here to continue:

- If your project is created by Teams Toolkit lower than v3.7.0, please create SPFx project with latest Teams Toolkit and migrate your codes.
- If your project is downloaded from Todo-list-SPFx sample app, please add the following configuration file to `SPFx` subfolder in your project:

```
{
  "@microsoft/generator-sharepoint": {
    "plusBeta": false,
    "isCreatingSolution": true,
    "version": ${SPFx_versioin},
    "libraryName": "todo-list-sp-fx",
    "libraryId": "c314487b-f51c-474d-823e-a2c3ec82b1ff",
    "environment": "spo",
    "packageManager": "npm",
    "solutionName": "todo-list-sp-fx",
    "solutionShortDescription": "todo-list-sp-fx description",
    "skipFeatureDeployment": true,
    "isDomainIsolated": false,
    "componentType": "webpart",
    "template": "react"
  }
}
```
Note: Replace `SPFx_version` with the SPFx version used in your project.


## 3. Failed to scaffold<a name="scaffold"></a>

### Error message
Project creation failed. A possible reason could be from Yeoman SharePoint Generator.

### Remediation
- Check SPFx development environment compatibility.    
  1. Check Node version by running the following command    
      ```
      node --version
      ```
  2. Check NPM version by running the following command
      ```
      npm --version
      ```
  3. Check whether the version of Node and NPM are compatibile with the latest SPFx according to [SharePoint Framework compatibility page](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/compatibility#spfx-development-environment-compatibility) and upgrade Node or NPM if needed.
- Or you could try to set up global SPFx development environment by following [Set up your SharePoint Framework development environment](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment#install-nodejs) and choose to scaffold using the globally installed packages.

## 2. Failed to import existing SPFx solution<a name="import"></a>

### Error message
Failed to retrieve existing SPFx solution information. Please make sure your SPFx solution is valid.

### Remediation

- Check your existing SPFx solution is valid with standard project folder structure. 
  1. Check your web part(s) is(are) located at `.\src\webparts` folder under your selected solution folder.

  2. Check your web part(s) manifest file is(are) located at `.\src\webparts\{webpartName}\{webpartName}WebPart.manifest.json`.

  3. Check your web part(s) manifest file has following properties:

     **id** - The property will be used as `entityId` to construct `staticTabs` in Teams manifest file.

     **preconfiguredEntries** - The property (_preconfiguredEntries[0].title.default_) will be used as `name` to construct `staticTabs` in Teams manifest file. 

- Or you could try to migrate your SPFx solution manually following [Integrate Teams Toolkit with an existing SPFx solution](https://github.com/OfficeDev/TeamsFx/wiki/Integrate-Teams-Toolkit-with-an-existing-SPFx-solution).

## 4. Failed to deploy<a name="deploy"></a>

### Error message
Failed to deploy due to lint errors in gulp bundle task. Example:

```
(×) Error: cli/runNpxCommand failed.
  (×) Error: Script ('npx gulp bundle --ship --no-color') execution error: Warning - lint - src/webparts/helloworld/HelloworldWebPart.ts(95,32): error @typescript-eslint/no-explicit-any: Unexpected any. Specify a different type. 
```

### Remediation

There's a known issue that deploy stage will fail even if there're only lint warnings in log detail. The root cause is that when there're lint errors in your SPFx project, `gulp bundle --ship --no-color` command in deploy stage will fail with exit code 1, but they're printed as warning in log details. See related [GitHub issue](https://github.com/SharePoint/sp-dev-docs/issues/9165) for more details.

You'll need to fix the lint errors or disable related lint rules to continue.

## 5. Add web part using Yeoman SharePoint generator of mismatched version<a name="addWebPart"></a>

To add additional web part in an existing SPFx solution, it is recommended to keep the version of @microsoft/generator that will be use to add web part and the solution version consistent to avoid any further build errors.

### Remediation
- Set up global SPFx dev dependencies     
  1. Check the version of your SPFx solution. You could find it from the value of "version" in `src/.yo-rc.json`.
  2. Install Yeoman SharePoint generator of your solution version
      ```
      npm install @microsoft/generator-sharepoint@{version} --global
      ```
      Note: if you don't have Yeoman installed before, you also need to install Yeoman following [Install Yeoman](https://learn.microsoft.com/en-us/sharepoint/dev/spfx/set-up-your-development-environment#install-yeoman)
  3. Use Teams Toolkit to add web part.
- Or you could continue adding SPFx web part using current Sharepoint generator package and upgrade SPFx solution after the web part is added.
  1. Choose "continue" when prompting to confirm whether to add web part using package of mismatched version.
  2. After the web part is added, you could upgrade your SPFx solution.
     1. Install CLI for Microsoft 365 following [CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/)
     2. Upgrade the project to the new version.
        1. You could find the new version from the value of dependencies (for example: the version of @microsoft/sp-core-library)in `src/package.json`.
        2. Run upgrade command.
            ```
            m365 spfx project upgrade --toVersion {version} --output md > "upgrade-report.md"
            ```  
            You could learn more about this command from [spfx project upgrade](https://pnp.github.io/cli-microsoft365/cmd/spfx/project/project-upgrade)     
        3. Follow the steps in the generated report to upgrade the project.






































































































