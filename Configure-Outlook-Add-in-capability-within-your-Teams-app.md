# Configure Outlook Add-in capability within your Teams app

## Introduction

Office Add-ins are web apps that extend the functionality of Outlook. Some of the things that can be done with an Outlook Add-in:

- Read and write the content of email messages and meeting invitations, as well as responses, cancellations, and appointments.
- Read properties of the user's mailbox.
- Respond automatically to events, such as the sending of an email.
- Integrate with external services including CRM and project management.
- Add custom ribbon buttons or menu items to perform specific tasks.

For more information, start with [Outlook Add-ins Overview](https://learn.microsoft.com/en-us/office/dev/add-ins/outlook/outlook-add-ins-overview).

## Prerequisites

To configure an Office Add-in as additional capability, you must meet the following conditions:

- You have a Microsoft 365 account to test the application. For example, an *.onmicrosoft.com account.
- Your Micrsoft 365 account has been added as an account in desktop Outlook. See [Add an email account to Outlook](https://support.microsoft.com/office/add-an-email-account-to-outlook-e9da47c4-9b89-4b49-b945-a204aeea6726)
- To deploy the Teams app to Azure as described in the last section of this article, you need an Azure account to an Azure subscription. Create your free Azure account if you don't already have one by using the link [Free Azure account](https://azure.microsoft.com/free/).

## Overview

The following are the major steps to adding an Outlook Add-in to a Teams app. Details are in the sections below.

1. [Prepare the Teams app project](prepare-the-teams-app-project)
1. [Create an Office Add-in project](#create-an-outlook-add-in-project) that is initially separate from your Teams app.
1. [Merge the manifest](#merge-the-manifest) from the Outlook Add-in project into the Teams app manifest.
1. [Copy the Outlook Add-in files to the Teams app project](#copy-the-outlook-add-in-files-to-the-teams-app-project).
1. [Edit the tooling configuration files](#edit-the-tooling-configuration-files).
1. [Move the application to Azure](#move-the-application-to-azure).

## Prepare the Teams app project

Begin by separating the source code for the tab (or bot) into its own subfolder. These instructions assume that the project initially has the following structure: 

```
|-- .vscode/
|-- appPackage/
|-- build\appPackage/
|-- env/
|-- infra/
|-- node_modules/
|-- public/
|-- src/
|-- gitignore
|-- package-lock.json
|-- package.json
|-- teamsapp.local.yml
|-- teamsapp.yml
|-- tsconfig.json
```

**NOTE**: If you are working with a new Teams tab project, the \node_modules folder and the package-lock.json file will not be present until you run `npm install` in the root of the project. The build\appPackage folder will not be present until after the first time you debug the project. 

1. Create a folder under the root named "tab" (or "bot").

    **NOTE**: For simplicity, the remainder of this article assumes that the existing Teams app is a tab. If you started with a bot instead, replace "tab" with "bot" in all of these instructions, including the content you add or edit in various files. 

1. Move the node_modules, public, and src folders into the new subfolder.
1. Move the package-lock.json, package.json, and tsconfig.json into the new subfolder. The project structure should now look like the following:

    ```
    |-- .vscode/
    |-- appPackage/
    |-- build\appPackage/
    |-- env/
    |-- infra/
    |-- tab/
    |-- |-- node_modules/
    |-- |-- public/
    |-- |-- src/
    |-- |-- package-lock.json
    |-- |-- package.json
    |-- |-- tsconfig.json
    |-- gitignore
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```

1. In package.json that you just moved to the tab folder, delete the script named "dev:teamsfx" from the "scripts" object. This script is added to a new package.json in the next step.
1. Create a new file named package.json *in the root of the project* and give it the following content:

    ```
    {
        "name": "CombinedTabAndAddin",
        "version": "0.0.1",
        "author": "Contoso",
        "scripts": {
            "dev:teamsfx": "env-cmd --silent -f .localSettings npm run start:tab",
            "build:tab": "cd tab && npm run build",
            "install:tab": "cd tab && npm install",
            "start:tab": "cd tab && npm run start",
            "test": "echo \"Error: no test specified\" && exit 1"
        },
        "devDependencies": {
            "@microsoft/teamsfx-run-utils": "alpha",
            "env-cmd": "^10.1.0"
        }
    }
    ```

1. Change the "name", "version", and "author" properties, as needed.
1. Open the teamsapp.local.yml file in the root of the project and find the line `args: install --no-audit`. Change this to `args: run install:tab --no-audit`.
1. Open **TERMINAL** in Visual Studio Code. Navigate to the root of the project and run `npm install`.
1. Verify that you can sideload the tab with the following steps:

    <ol type="a">
      <li>Select <b>View</b> | <b>Run</b> in Visual Studio Code.</li>
      <li>In the <b>RUN AND DEBUG</b> drop down menu, select the top option, <b>Debug (Edge)</b>, and then press F5. The project will build and run. Among other things, a new node_modules folder will be created in the project root. This process may take a couple of minutes. Eventually, Teams opens in a browser with a prompt to add your tab app.</li>
      <li>Select <b>Add</b>.</li>
      <li>To stop debugging and uninstall the app, select <b>Run</b> | <b>Stop Debugging</b> in Visual Studio Code.</li>
    </ol>

## Create an Outlook Add-in project

1. Open a second instance of Visual Studio Code.
1. With Teams Toolkit open in Visual Studio Code, select **Create a new app**.
1. In the **Select an option** drop down, select **Start with an Outlook add-in**, and then select **Outlook Taskpane Add-in (preview)**.
1. Give a name (with no spaces) to the project when prompted and Teams Toolkit will create the project with basic files and scaffolding *and open it in a separate Visual Studio Code window*. You will used this project as a source for files and markup that you add to the Teams project.
1. Although you won't be developing this project, verify that it can be sideloaded to from Visual Studio Code works before you continue. Use the following steps:

    <ol type="a">
      <li><i>First, make sure Outlook desktop is closed.</i></li>
      <li>In Visual Studio Code, open the Teams Toolkit.</li>
      <li>In the <b>ACCOUNTS</b> section, verify that you are signed into Microsoft 365.</li>
      <li>Select <b>View</b> | <b>Run</b> in Visual Studio Code. In the <b>RUN AND DEBUG</b> drop down menu, select the option, <b>Outlook Desktop (Edge Chromium)</b>, and then press F5. The project builds and a Node dev-server window opens. This process may take a couple of minutes. Eventually, Outlook desktop will open.</li>
      <li>Open the <b>Inbox</b> <i>of your Microsoft 365 account identity</i> and open any message. A <b>Contoso Add-in</b> tab with two buttons will appear on the <b>Home</b> ribbon (or the <b>Message</b> ribbon, if you have opened the message in its own window).</li>
      <li>Click the <b>Show Taskpane</b> button and a task pane opens. Click the <b>Perform an action</b> button and a small notification appears near the top of the message.</li>
      <li>To stop debugging and uninstall the add-in, select <b>Run</b> | <b>Stop Debugging</b> in Visual Studio Code.</li>
    </ol>

## Merge the manifest

The Teams app's manifest is generated at debug-and-sideload time (or build time) from the file manifest.json in the \appPackage folder of the Teams project. *This file is actually a kind of **template** for a manifest. In this article it is referred to as the "template" or "manifest template".* Most of the markup is hardcoded into the template, but there are also some configuration files that contain data that gets injected into the final generated manifest. In this procedure, you do the following:

- Copy markup from the add-in's manifest to the Teams app's manifest template.
- Edit the configuration files. 

Unless specified otherwise, the file you change is \appPackage\manifest.json.

1. Copy the "$schema" and "manifestVersion" property values from the add-in's manifest to the corresponding properties of the Teams app's manifest.template.json file.
1. Adjust the "name.full", "description.short", and "description.full" property values as needed to take account of the fact that an Outlook add-in is part of the app. 
1. Do the same for the "name.short" value. We recommend that you keep the `${{TEAMSFX_ENV}}` on the end of the name. This variable is replaced with "local" in dev mode and with "dev" in production mode.

    The following is an example:

    ```
    "short": "Fabrikam Tab and Add-in-${{TEAMSFX_ENV}}",
    ```

    **Notes**:
    The "name.short" value appears in both the Teams tab app and the Outlook add-in. Examples: 

    - It is the label under the launch button of the tab app.
    - It is content of the title bar of the add-in's task pane.

1. If you changed the "name.short" value from its default (which is the name of the project followed by the `${{TEAMSFX_ENV}}` variable), make exactly the same change in all places where the project name appears in the following two files in the root of the Teams project: teamsapp.yml and teamsapp.local.yml.
1. If there is no "authorization.permissions.resourceSpecific" array in the Teams manifest template, copy it from the add-in manifest as a top-level property. If there already is one in the Teams template, copy any objects from the array in the add-in manifest to the array in the Teams template. The following is an example:

    ```json
    "authorization": {
        "permissions": {
            "resourceSpecific": [
                {
                    "name": "MailboxItem.Read.User",
                    "type": "Delegated"
                }
            ]
        }
    },
    ```

1. In the .env.local file, find the lines that assign values to the `TAB_DOMAIN` and `TAB_ENDPOINT` variables. Add the following lines immediately below them:

    ```
    ADDIN_DOMAIN=localhost:53000
    ADDIN_ENDPOINT=https://localhost:53000
    ```

    **NOTE**: During the early preview phase, this variables are set to the same string as the two `TAB_...` variables. It is not possible at this time to host the add-in and the Teams app on distinct domains.

1. Add the following line to the end of the .env.dev file:

    ```
    ADDIN_ENDPOINT=
   ```
   
1. In the Teams manifest template, add the placeholder `"${{ADDIN_DOMAIN}}",` to the top of the "validDomains" array. The Teams Toolkit will replace this with a localhost domain when you are developing locally. When you deploy the finished Teams app to staging or production as described below in [Move the application to Azure](#move-the-application-to-azure), Teams Toolkit will replace the placeholder with the staging/production URI. The following is an example:

    ```json
    "validDomains": [
        "${{ADDIN_DOMAIN}}",
        
        // other domains or placeholders
    ],
    ```

1. Copy the entire "extensions" property from the add-in's manifest into the Teams app manifest template as a top-level property.
1. In the Teams app manifest template, replace every occurrence of "localhost:3000" with "localhost:53000".

## Copy the Outlook Add-in files to the Teams app project

1. Create a top-level folder called "add-in" in the Teams app project.
1. Copy the following files and folders from the add-in project to the "add-in" folder of the Teams app project.

    - /assets
    - /src
    - .eslintrc.json
    - babel.config.json
    - package-lock.json
    - package.json
    - tsconfig.json
    - webpack.config.js

    **NOTE**: Do *not* copy over the manifest.json file.

    Your folder structure should now look like the following:

    ```
    |-- .vscode/
    |-- add-in/
    |-- |-- assets/
    |-- |-- src/
    |-- |-- |-- commands/
    |-- |-- |-- taskpane/
    |-- |-- .eslintrc.json
    |-- |-- babel.config.json
    |-- |-- package-lock.json
    |-- |-- package.json
    |-- |-- tsconfig.json
    |-- |-- webpack.config.js
    |-- appPackage/
    |-- build\appPackage/
    |-- env/
    |-- infra/
    |-- node_modules/
    |-- tab/
    |-- |- node_modules/
    |-- |-- public/
    |-- |-- src/
    |-- |-- package-lock.json
    |-- |-- package.json
    |-- |-- tsconfig.json
    |-- gitignore
    |-- package.json
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```

## Edit the tooling configuration files

1. Open the package.json file *in the root of the Teams app*.
1. Add the following scripts to the "scripts" object:

    ```
    "install:add-in": "cd add-in && npm install",
    "build:add-in": "cd add-in && npm run build",
    "build:add-in:dev": "cd add-in && npm run build:dev",
    "build": "build:add-in && build:tab",
    "postinstall": "install:add-in && install:tab"
    ```

    **NOTE**: There is no combined "start" script parallel to the combined "build" and "postinstall" scripts because simultaneous debugging of the tab app and the add-in is not currently supported.

1. Open the package.json file *in the add-in folder* (not the tab folder, and not the root of the Teams app). 
1. In the "config" object, change the "dev_server_port" value to `53000`.
1. Several of the scripts in the "scripts" object have a `manifest.json` parameter like the following. 

    ```
    "start": "office-addin-debugging start manifest.json",
    "start:desktop": "office-addin-debugging start manifest.json desktop",
    "start:web": "office-addin-debugging start manifest.json web",
    "stop": "office-addin-debugging stop manifest.json",
    "validate": "office-addin-manifest validate manifest.json",
    ```

    In the "start", "start:desktop", and "start:web" scripts, change `manifest.json` to `../build/AppPackage/appPackage.local.zip`. When you are done, they should look like this:

    ```
    "start": "office-addin-debugging start ../build/AppPackage/appPackage.local.zip",
    "start:desktop": "office-addin-debugging start ../build/AppPackage/appPackage.local.zip desktop",
    "start:web": "office-addin-debugging start ../build/AppPackage/appPackage.local.zip web",
    ```

    In the "validate" and "stop" scripts, change the parameter to `../build/AppPackage/manifest.local.json`. When you are done, they should look like this:

    ```
    "stop": "office-addin-debugging stop ../build/AppPackage/manifest.local.json",
    "validate": "office-addin-manifest validate ../build/AppPackage/manifest.local.json",
    ```

1. In Visual Studio Code, open the **TERMINAL**. Navigate to the add-in folder, then run the command `npm install`. 
1. In the add-in folder, open the webpack.config.js file. Change the line `from: "manifest*.json",` to `from: "../build/appPackage/manifest*.json",`.
1. Near the end of the webpack.config.js file there is a line that assigns a port for the webpack dev server. Change the value from `3000` to `53000`.
1. In the root of project, open the teamsapp.local.yml file and find the `configureApp` section. Use the `#` character to comment out the lines that validate the manifest template. This is necessary because the Teams manifest validation system is not yet compatible with the changes you made to the manifest template. When you are done, the `configureApp` section should begin like the following:

    ```
    configureApp:
      - uses: file/updateEnv # Generate env to .env file
        with:
          envs:
            TAB_DOMAIN: localhost:53000
            TAB_ENDPOINT: https://localhost:53000
    #  - uses: teamsApp/validate
    #    with:
    #      manifestPath: ./appPackage/manifest.template.json # Path to manifest template
    
    # remainder of the section omitted
    
   ```

1. Open the .vscode\tasks.json file in the add-in project and copy all of the tasks in the "tasks" array. *Add* them to "tasks" array of the same file in the Teams project. *Do not remove any of the tasks that are already there.* Be sure all tasks are separated by commas. 
1. In *each* of the task objects that you just copied, add the following "options" property to ensure that these tasks run in the add-in folder.

    ```
    "options": {
        "cwd": "${workspaceFolder}/add-in/"
    },
    ```

    For example, the "Debug: Outlook Desktop" task should like the following when you are done. 

    ```
    {
        "label": "Debug: Outlook Desktop",
        "type": "npm",
        script": "start:desktop -- --app outlook",
        "presentation": {
            "clear": true,
            "panel": "dedicated",
        },
        "problemMatcher": [],
        "options": {
            "cwd": "${workspaceFolder}/add-in/"
        }
    }
    ```

1. Add the following task to the "tasks" array in the .vscode\tasks.json file of the Teams project. Note the following about this markup: 

    - It adds a "Start Add-in Locally" task that combines the tab app's "Create resources" task with the add-in's debugging task and specifies that they must run in that order.
    - The "Create resources" task generates the final manifest.

    ```
    {
        "label": "Start Add-in Locally",
        "dependsOn": [
            "Create resources",
            "Debug: Outlook Desktop"
        ],
        "dependsOrder": "sequence"
    },
    ```

1. Open the .vscode\launch.json file in the Teams project, which configures the **RUN AND DEBUG** UI in Visual Studio Code and add the following object to the top of the "configurations" array.

    ```
    {
        "name": "Outlook Desktop (Edge Chromium)",
        "type": "msedge",
        "request": "attach",
        "port": 9229,
        "timeout": 600000,
        "webRoot": "${workspaceRoot}/add-in/",
        "preLaunchTask": "Start Add-in Locally",
        "postDebugTask": "Stop Debug"
    },
    ```

1. Verify that you can sideload the add-in part of the Teams app to Outlook with the following steps:

    <ol type="a">
      <li><i>First, make sure Outlook desktop is closed.</i></li>
      <li>In Visual Studio Code, open the Teams Toolkit.</li>
      <li>In the <b>ACCOUNTS</b> section, verify that you are signed into Microsoft 365.</li>
      <li>Select <b>View</b> | <b>Run</b> in Visual Studio Code. In the <b>RUN AND DEBUG</b> drop down menu, select the option, <b>Outlook Desktop (Edge Chromium)</b>, and then press F5. The project builds and a Node dev-server window opens. This process may take a couple of minutes. Eventually, Outlook desktop will open.</li>
      <li>Open the <b>Inbox</b> <i>of your Microsoft 365 account identity</i> and open any message. A <b>Contoso Add-in</b> tab with two buttons will appear on the <b>Home</b> ribbon (or the <b>Message</b> ribbon, if you have opened the message in its own window).</li>
      <li>Click the <b>Show Taskpane</b> button and a task pane opens. Click the <b>Perform an action</b> button and a small notification appears near the top of the message.</li>
      <li>To stop debugging and uninstall the add-in, select <b>Run</b> | <b>Stop Debugging</b> in Visual Studio Code.</li>
    </ol> 

## Move the application to Azure

1. Open the teamsapp.yml file in the root of the project. Replace the entire `deploy:` section with the following code. These changes take account of the new folder structure and the fact that both add-in and tab files need to be deployed.

    ```
    deploy:
      - name: npmInstallTab
        uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./tab
          args: install
      - name: buildTab
        uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./tab
          args: run build --if-present
      - name: npmInstallAddin
        uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./add-in
          args: install
      - name: buildAddin
        uses: cli/runNpmCommand # Run npm command
        with:
          workingDirectory: ./add-in
          args: run build --if-present # The add-in's build will copy add-in's dist folder to tab's build folder, so both tab and add-in are deployed.

      - uses: azureStorage/deploy # Deploy bits to Azure Storage Static Website
        with:
          workingDirectory: .
          distributionPath: ./tab/build # Deploy base folder
          resourceId: ${{TAB_AZURE_STORAGE_RESOURCE_ID}} # The resource id of the cloud resource to be deployed to
    ```

1. In Visual Studio Code open the Teams Toolkit and in the **ACCOUNTS** section be sure you are signed into your Azure account. For more information about signing in, open [Exercise - Create Azure resources to host a Teams tab app](https://learn.microsoft.com/training/modules/teams-toolkit-vsc-deploy-apps/03-create-azure-resources-exercise) and scroll to the **Sign in to Azure in Teams Toolkit** section.
1. In the **DEPLOYMENT **section of Teams Toolkit, select **Provision in the cloud**. It may take several minutes.
1. When provisioning completes, select **Deploy to the cloud** to deploy your app code to Azure.
1. To run the tab capability from the remote deployment, select **View**| **Run** in Visual Studio Code and in the drop down, select one of the following:

    - **Launch Remote (Edge)**
    - **Launch Remote (Chrome)** 

    Press F5 to preview your Teams app.

1. To run the add-in capability from the remote deployment, ... ?

Copy the production URL from the TAB_ENDPOINT in env/.env.dev file.
Edit webpack.config.js file and change urlProd to the value you just copied. Please note to add a '/' at the end of the URL.
Run npm run build:add-in.
Copy add-in\dist\manifest.dev.json to appPackage folder using npx ncp .\add-in\dist\manifest.dev.json .\appPackage\manifest.addinPreview.json
Run npx office-addin-dev-settings sideload .\appPackage\manifest.addinPreview.json

## What’s next

There are other commonly suggested next steps, for example:

- [Add authentication and make a Graph API call](https://learn.microsoft.com/microsoftteams/platform/toolkit/add-single-sign-on?pivots=visual-studio-code)
- [Set up CI/CD pipelines](https://github.com/OfficeDev/TeamsFx/wiki/How-to-automate-cicd-pipelines)
- [Call a backend API](https://github.com/OfficeDev/TeamsFx/wiki/How-to-integrate-Azure-Functions-with-your-Teams-app)
