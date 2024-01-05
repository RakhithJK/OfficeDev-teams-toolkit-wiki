> The content is under construction and is subject to rapid changes.

# teamsfx preview
> Before running teamsfx preview, teamsfx provision and teamsfx deploy should be run first.

Preview the current application from local or remote.

## Parameters for teamsfx preview
|Parameter|Requirement|Description|
|--|--|--|
|`--env`|No|Select an existing env for the project. The default value is `local`.|
|`--run-command`|No|The command to start local service. Work for `local` environment only. If not specified, teamsfx will use the auto detected one from project type (`npm run dev:teamsfx` or `dotnet run` or `func start`). If empty, teamsfx will skip starting local service.|
|`--running-pattern`|No|The ready signal output that service is launched. Work for `local` environment only. If not specified, teamsfx will use the default common pattern (`started\|successfully\|finished\|crashed\|failed`). If empty, teamsfx treats process start as ready signal.|
|`--open-only`|No|Work for `local` environment only. If true, directly open web client without launching local service.|
|`--m365-host`|No|Preview the application in Teams, Outlook or the Microsoft 365 app. Options are `teams`, `outlook` and `office`. The default value is `teams`.|
|`--browser`|No|The browser to open Teams web client. The options are `chrome`, `edge` and `default` such as system default browser and the value is `default`.|
|`--browser-arg`|No|Argument to pass to the browser, requires --browser, can be used multiple times, for example, `--browser-args="--guest"`|
|`--exec-path`| No | The paths that will be added to the system environment variable PATH when the command is executed, defaults to `${folder}/devTools/func`. |

## Scenarios for teamsfx preview
The following list provides the common scenarios for`teamsfx preview:
- Local Preview Tab App
  - Executing the commands in your project directory.
    ```shell
    teamsfx provision --env local
    teamsfx deploy --env local
    teamsfx preview --env local
    ```

- Local Preview Bot App / Message Extension App
  - Install [dev tunnel cli](https://aka.ms/teamsfx-install-dev-tunnel).
  - Login with your M365 Account using the command `devtunnel user login`.
  - Start your local tunnel service by running the command `devtunnel host -p 3978 --protocol http --allow-anonymous`.
  - In the `env/.env.local` file, fill in the values for `BOT_DOMAIN` and `BOT_ENDPOINT` with your dev tunnel URL.
    ```
    BOT_DOMAIN=sample-id-3978.devtunnels.ms
    BOT_ENDPOINT=https://sample-id-3978.devtunnels.ms/
    ```
  - Executing the commands in your project directory.
    ```shell
    teamsfx provision --env local
    teamsfx deploy --env local
    ```
  - If you are previewing a Azure Functions hosted notification bot, please execute the following command in your project directory.
    ```shell
    npm run prepare-storage:teamsfx
    ```
  - Executing the command in your project directory.
    ```shell
    teamsfx preview --env local
    ```

- Preview Bot App in Teams App Test Tool
  - For JavaScript / TypeScript projects, you can run the following commands in project directory:
    - Install [Teams App Test Tool](https://www.npmjs.com/package/@microsoft/teams-app-test-tool) CLI.
      ```
      npm install -g @microsoft/teams-app-test-tool
      ```
    - Execute the deploy command to install required dependencies and npm packages.
      ```
      teamsfx deploy --env=testtool
      ```
    - Execute the command to start your bot app.
      ```
      npm run dev:teamsfx:testtool
      ```
    - Execute the command in another terminal to start Teams App Test Tool.
      ```
      npm run dev:teamsfx:launch-testtool
      ```
    - A browser will pop up to open Teams App Test Tool and you can test your bot in it.

    > If you want to use a specific version of Teams App Test Tool, you can download it manually and add to the PATH environment variable. The `npm run dev:teamsfx:launch-testtool` script will try to find `teamsapptester` command from `PATH`.
    
    > If you changed your bot app's port number or message endpoint to one that differs from the default `http://127.0.0.1:3978/api/messages`, you need to set `BOT_ENDPOINT` environment variable in `.env.testtool` so that the test tool can connect to your bot app. For example:
    > ```
    > BOT_ENDPOINT=http://127.0.0.1:6978/my/message/endpoint
    > ```
    
    > If you failed to start test tool because of port conflict, you can change the test tool's port number by setting the `TEAMSAPPTESTER_PORT` environment variable in `.env.testtool`. For example:
    > ```
    > TEAMSAPPTESTER_PORT=56150
    > ```

  - For C# projects, you can run the following commands in the directory that contains `teamsapp.yml` file. This is usually in a subfolder named the same as your project name.
    - Download Teams App Test Tool CLI from [GitHub release](https://github.com/OfficeDev/TeamsFx/releases?q=teams-app-test-tool&expanded=true) and unzip the downloaded package to a folder, for example, `C:\teams-app-test-tool`, and you can see an exe binary file named `teamsapptester.exe`.
    - Execute the command to start your bot app.
      ```
      dotnet run --launch-profile "Teams App Test Tool (browser)"
      ```
    - Specify your bot message endpoint with `BOT_ENDPOINT` environment variable and execute the command in another terminal to start Teams App Test Tool.
      - For Command Prompt:
        ```cmd
        set BOT_ENDPOINT=http://127.0.0.1:5130/api/messages
        C:\teams-app-test-tool\teamsapptester.exe start
        ```

      - For Power Shell
        ```powershell
        $env:BOT_ENDPOINT = "http://127.0.0.1:5130/api/messages"
        C:\teams-app-test-tool\teamsapptester.exe start
        ```
      - A browser will pop up to open Teams App Test Tool and you can test your bot in it.

      > If you failed to start test tool because of port conflict, you can change the test tool's port number by setting the `TEAMSAPPTESTER_PORT` environment variable before running `teamsapptester.exe` command.

- Remote Preview
```shell
teamsfx provision --env dev
teamsfx deploy --env dev
teamsfx preview --env dev
```
