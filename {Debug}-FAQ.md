# Teams Toolkit Debug FAQ

## Overall
Teams Toolkit allows you to debug your Teams app locally by leveraging Visual Studio Code debugging features. After pressing F5, several components of the app will be automatically started. The Teams web client will then be launched in your browser. Specifically, the following components may be started according to your app capabilities:
- Tab: a react app required by Teams Tab capability
- Function: a Azure Functions app that may be needed by Tab
- Bot: a bot server required by Teams Bot capability
- Dev Tunnel: a tunneling service required by Teams Bot that forwards local address to public address

During debugging, a localhost development certificate will also be automatically generated and installed to your system after your confirmation.

Some frequently asked questions are listed bellow.

## Which ports will be used?
| Component | Port |
| --- | --- |
| Tab | 53000, or 3000 (for Teams Toolkit version < 3.2.0) |
| Function | 7071 |
| Node inspector for Function | 9229 |
| Bot / Messaging Extension | 3978 |
| Node inspector for Bot / Messaging Extension | 9239 |

## What to do if some port is already in use?

### Error
![Port Already In Use](debug/port-already-in-use.png)

### Reason
This is mainly because this port was not successfully closed after last debug.

### Mitigation
You can follow the scripts below to find the process that occupies this port, and to kill that process. After the process is killed, start debugging again.

For Windows, in cmd or powershell:
```cmd
> netstat -ano | findstr <port>
> tskill <process id>
```

For Linux or OSX, in shell:
```shell
$ lsof -i:<port>
$ kill <process id>
```

## What to do if I want to add my own environment variables for debugging?
Teams Toolkit scaffolded projects come with 2 predefined environments, local and dev. We will use local for the debug process. More about [environments](https://aka.ms/teamsfx-v5.0-guide#environments)

Update `env/.env.local` or `env/.env.local.user` with your own environment variables for debugging.

## What to do if I want to use my own tunneling service instead of the built-in one for Bot or Messaging Extension?
### Reason
Since Bot and Messaging Extension requires a public address as the messaging endpoint, dev tunnel will be used by default to automatically create a tunnel connection forwarding localhost address to public address.
You can follow [Choose your own tunnel solution](https://github.com/OfficeDev/TeamsFx/wiki/%7BDebug%7D-Teams-Toolkit-v5-VS-Code-Tasks#choose-your-own-tunnel-solution).

## What to do if Teams shows "App not found" when the Teams web client is opened?
### Error

![App Not Found](debug/app-not-found.png)

### Reason

This is mainly because the Teams account you logged in when the Teams web client is opened is different from the M365 account you logged in when developing the Teams app.

### Mitigation
Please make sure you use the same M365 account. After logging in the correct account, start debugging again. You can see which M365 account you logged in via Teams Toolkit, like:

![Teams Toolkit M365 Account](debug/m365-account.png)

## What to do if Teams shows "Something went wrong" when the Teams web client is opened?
### Error

![Something Went Wrong](debug/something-went-wrong.png)

### Reason
This is mainly because there is some error in manifest.

### Mitigationn
Please [open an issue](https://github.com/OfficeDev/TeamsFx/issues/new/choose) with enough context and information.

## What to do if Teams shows "Permission needed" when the Teams web client is opened?
### Error

![Permission Needed](debug/permission-needed.png)

### Reason

This is mainly because the custom app uploading is not turned on for your Teams tenant.

### Mitigation
You can follow [this document](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant#enable-custom-teams-apps-and-turn-on-custom-app-uploading) to turn it on.

## What to do if I do not want to install the development certificate?
### Reason
Since Teams requires https Tab hosting endpoint, a localhost development certificate will be automatically generated and installed to your system after your confirmation. The confirmation window will be popped up during debugging, like:

![Install-Certificate-Confirmation](debug/install-certificate-confirmation.png)

### Mitigation
We recommend you to install the development certificate. However, if you do not want to install the development certificate and do not want the confirmation window to pop up every time during debugging, you can follow the script bellow to disable the development certificate.

Close the trust development certificate setting, then start debugging.

For VSCode, you should set the setting `fx-extension.prerequisiteCheck.devCert` to be false.
![VSCode trust dev cert](debug/vsc-trust-dev-cert.jpg)
For CLI, you should run command `teamsfx config set trust-development-certificate off`.

If so, an error will show in the Tab page of your app, look like:

![Tab-Https-Not-Trusted](debug/tab-https-not-trusted.png)

To resolve this issue, open a new tab in the same browser, go to https://localhost:53000/index.html#/tab, click the "Advanced" button and then select "Proceed to localhost (unsafe)". After doing this, refresh the Teams web client.

![Continue-To-Localhost](debug/continue-to-localhost.png)

## How to manually install the development certificate for Windows Subsystem for Linux (WSL) users?
### Reason
Since Teams requires https Tab hosting endpoint, a localhost development certificate will be automatically generated when you launch debug. Teams toolkit runs on WSL but the browser runs on Windows, so the dev certificate will not be automatically installed. If the development certificate is not installed, debug will fail after adding app to Teams.

![Tab-Https-Not-Trusted](debug/tab-https-not-trusted.png)

### Mitigation
#### Method 1: Trust the development certificate in browser
This method is simpler but only takes effect for current browser. You need to repeat these steps for each browser you use to debug your app.

1. Open a new tab in the same browser, go to https://localhost:53000/index.html#/tab.
2. Click the "Advanced" button and then select "Proceed to localhost (unsafe)".
3. Refresh the Teams web client.

![Continue-To-Localhost](debug/continue-to-localhost.png)

#### Method 2: Trust the development certificate in Windows
This method is a little bit more complex but it takes effect globally. You only need to do once for all browsers.

1. Open the certificate folder of your WSL distribution in Windows Explorer (example path: `\\wsl$\{DISTRO_NAME}\home\{USER_NAME}\.fx\certificate`).

    ![WSL-Cert-Folder](debug/wsl-cert-1-folder.png)

2. Open "localhost.crt" and click "Install Certificate...".

    ![WSL-Cert-Localhost-Crt](debug/wsl-cert-2-localhostcrt.png)

3. In the "Certificate Import Wizard", select "Next".

    ![WSL-Cert-Import-Wizard](debug/wsl-cert-3-import-wizard.png)

4. Select "Place all certificates in the following store" and click "Browse".

    ![WSL-Cert-Browse](debug/wsl-cert-4-browse.png)

5. Select "Trusted Root Certification Authorities", click "OK" and then click "Next".

    ![WSL-Cert-Root-Cert](debug/wsl-cert-5-root-cert.png)

6. Click "OK" to confirm importing the certificate.

    ![WSL-Cert-Confirm](debug/wsl-cert-6-confirm.png)

7. You will see a confirmation that the import process has succeeded.

    ![WSL-Cert-Succeed](debug/wsl-cert-7-succeed.png)

8. Restart your browser to take effect.

## SPFx known issue on Teams workbench debug on macOS/Linux
### Error
![Error loading debug manifests](debug/error-loading-debug-manifests.png)
### Reason
For SPFx project, our toolkit will also help install the development certificate but it may be invalid on macOS/Linux system, thus on Teams workbench debug, it will fail to connect the debug manifest url.
### Mitigation
To resolve this issue, open a new tab in the same browser, go to https://localhost:4321/temp/manifests.js, click the "Advanced" button and then select "Proceed to localhost (unsafe)". After doing this, refresh the Teams web client.

![Continue To SPFx Localhost](debug/continue-to-spfx-localhost.png)

## Error "Value cannot be null. (Parameter 'provider')" when starting bot/api project
### Error
![Func Error](debug/func-value-cannot-be-null-error.png)
### Reason
Azure Functions Core Tools will download required dependencies on the first execution. This error occurs when there's failure during the dependency downloading period.
### Mitigation
To resolve this issue, ensure your network connection is stable when launching local service, and try again.

If still fail with the same error, try:
- Set Azure Functions log level to "*Debug*" to get more detailed logs when starting.
  ``` json
  // host.json of your azure functions
  {
    "version": "2.0",
    "logging": {
      ...
      "logLevel": {
        "default": "Debug"
      }
    },
    ...
  }
  ```
- Clear Azure Functions Core Tools local cache at `${HOME}/.azure-functions-core-tools/`.
