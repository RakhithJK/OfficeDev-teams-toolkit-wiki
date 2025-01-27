# Teams Toolkit v5 VS Code Tasks

A Teams Toolkit generated project has pre-defined a set of VS Code tasks in its `.vscode/tasks.json`. These tasks are for debugging and have corresponding arguments as inputs. This page shows the details about how these tasks are defined and how to customize with your own args.

> Note: these tasks are generated by Teams Toolkit (>= 5.0.0). To use those tasks generated by Teams Toolkit (>= 4.1.0), see [{Debug} Teams Toolkit VS Code Tasks](https://github.com/OfficeDev/TeamsFx/wiki/%7BDebug%7D-Teams-Toolkit-VS-Code-Tasks) page.

## Overall Contract

Aligning with the official VS Code schema, the `tasks.json` contains **`tasks`** top-level properties, which defines all the tasks and execution sequence used by Teams Toolkit. Following shows the summary of those pre-defined tasks:

| Task Label<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*(Type:Command)*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Description |
|-----|-----|
| **Start Teams App Locally** | The entry task of debugging, to be referenced by `launch.json`. |
| **Validate prerequisites**<br>(*teamsfx:debug-check-prerequisites*) | Check prerequisites required by debugging. |
| **Start local tunnel**<br>(*teamsfx:debug-start-local-tunnel*) | Start local tunneling for bot project. |
| **Provision**<br>(*teamsfx:provision*)| Create Teams app related resources required by debugging. |
| **Deploy**<br>(*teamsfx:deploy*)| Build project. |
| **Start application**<br>(*shell:npm run ...*) | Launch all local services. |
| **Launch desktop client** | Launch Teams desktop client. |

> Note: Depend on your project type, your `tasks.json` may contain a subset of above tasks.
>
> For customization, all *teamsfx:* tasks contain `args` as inputs. See each section of task definition about the `args` schema.

## Task Definition
This sections shows the details of each task.

### Start Teams App Locally

This task is the entry point of all other tasks. It represents the full flow to launch Teams app locally. If you'd like to skip any step(s), just comment it out, e.g.,

```json
{
    "label": "Start Teams App Locally",
    "dependsOn": [
        //// comment out any step(s) you'd like to skip
        // "Validate prerequisites",
        "Start local tunnel",
        "Provision",
        "Deploy",
        "Start application"
    ],
    "dependsOrder": "sequence"
}
```

### Validate prerequisites

This task is to validate prerequisites that will be used in following debugging steps. If you'd like to skip checking any prerequisite(s), just comment it out.

| Arguments       | type   | required | description                                                                                                                               |
|-----------------|--------|----------|------------------------------------------------------------------------------------------------------------|
| prerequisites   | array  | required | The enabled prerequisite checkers. Checkers: nodejs, m365Account, devCert, portOccupancy |
| portOccupancy   | array  | required | The ports to check if they are in use.                                                                    |

| prerequisites  | description |
|----------------|------------------------------------------------------------------------------------------------------------|
| nodejs | Validate if Node.js is installed. |
| m365Account | Sign-in prompt for Microsoft 365 account, then validate if the account has the custom app upload permission. |
| portOccupancy | Validate available ports to ensure those debugging ones are not occupied. |

#### Sample

```json
{
    "label": "Validate prerequisites",
    "type": "teamsfx",
    "command": "debug-check-prerequisites",
    "args": {
        "prerequisites": [
            //// comment out any checker(s) you'd like to skip
            // "nodejs",
            "m365Account",
            "portOccupancy"
        ],
        "portOccupancy": [
            3978,
            //// add or update your own port(s) to be validated
            // 9239,
            2233
        ]
    }
},
```
#### Node.js
Teams Toolkit includes a node.js prerequisite check that verifies whether the version of Node.js installed in your development environment exists and meets the requirements in the `engines.node` field of the project's `package.json` file.

When you create a project using Teams Toolkit, it is fully tested and validated using the [LTS version](https://nodejs.dev/en/about/releases/) of Node.js that was available at the time of creation. The supported Node.js versions for your project are specified in the `engines.node` field of your `package.json` file.

If your project has been tested and works correctly with a different version of Node.js, you can update the `engines.node` field in your package.json file to reflect the supported version. Doing so will remove the warning generated by the prerequisite check.

#### Port Occupancy
For newly created project, there may be following ports to be validated:
- 53000: the default port for local tab service
- 3978: the default port for local bot service
- 9239: the default debugger port for local bot service
- 7071: the default port for local api/backend service
- 9229: the default debugger port for local api/backend service
- 4321: the default port for local SPFx service

### Start local tunnel

This task is to start local tunnel service to make your local bot message endpoint public.

#### Dev Tunnel
If you choose to use the dev tunnel service, the following are the required arguments:

| Arguments | Type   | Required | Description |
|-----------|--------|----------|-------------|
| type      | string | required | The type of tunnel service to use. This argument must be set to `dev-tunnel`. |
| env       | string | optional | The environment name. Teams Toolkit will write the environment variables defined in `output` to `.env.<env>` file. |
| ports     | array  | required | An array of port configurations, each specifying the local port number, protocol, and access control settings. |

The `ports` argument must be an array of objects, with each object specifying the configuration for a particular port.
Each object must have the following fields:

| Port                   | Type   | Required | Description |
|------------------------|--------|----------|-------------|
| portNumber             | number | required | The local port number of the tunnel. |
| protocol               | string | required | The protocol of the tunnel. |
| access                 | string | optional | The access control setting for the tunnel. This value can be set to `private` or `public`. If not specified, the default value is `private`. |
| writeToEnvironmentFile | object | optional | The key of tunnel endpoint and tunnel domain environment variables that will be writen to `.env` file. |

The `writeToEnvironmentFile`object may have two fields:
| WriteToEnvironmentFile | Type   | Required | Description |
|------------------------|--------|----------|-------------|
| endpoint               | string | optional | The key of tunnel endpoint environment variable.|
| domain                 | string | optional | The key of tunnel domain environment variable.|

When `writeToEnvironmentFile` is included, the specified environment variables will be written to the `.env` file. If this field is omitted, no environment variables will be written to the file. 

#### Dev Tunnel Samples
**1. The default one used by TeamsFx templates.** If you want to manually migrate your local tunnel task from a v4 project, you can use the following code to replace the old task.
```json
{
    "label": "Start local tunnel",
    "type": "teamsfx",
    "command": "debug-start-local-tunnel",
    "args": {
        "type": "dev-tunnel",
        "ports": [
            {
                "portNumber": 3978,
                "protocol": "http",
                "access": "public",
                "writeToEnvironmentFile": {
                    "endpoint": "BOT_ENDPOINT",
                    "domain": "BOT_DOMAIN"
                }
            }
        ],
        "env": "local"
    },
    "isBackground": true,
    "problemMatcher": "$teamsfx-local-tunnel-watch"
},
```

**2. Change port.** To use another port for local bot service (e.g., 3922), you can change the one in `portNumber`. Note that you also need to change the port in bot code (`index.js` or `index.ts`).
```json
{
    "label": "Start local tunnel",
    "type": "teamsfx",
    "command": "debug-start-local-tunnel",
    "args": {
        "type": "dev-tunnel",
        "ports": [
            {
                "portNumber": 3922,
                "protocol": "http",
                "access": "public",
                "writeToEnvironmentFile": {
                    "endpoint": "BOT_ENDPOINT",
                    "domain": "BOT_DOMAIN"
                }
            }
        ],
        "env": "local"
    },
    "isBackground": true,
    "problemMatcher": "$teamsfx-local-tunnel-watch"
},
```
#### Choose your own tunnel solution.

If your dev environment does not support Teams Toolkit dev tunnel or you prefer to use a different tunnel solution, you can choose from a variety of options including:

|Alternative|Description|
|-|-|
|[ngrok](https://ngrok.com/)| An alternative tunnel solution. To simplify the process of debugging your Teams project using ngrok, you can follow these [detailed instruction](https://aka.ms/teamsfx-tasks/customize-tunnel-service).|
|[localtunnel](https://localtunnel.me/)|An alternative tunnel solution. You can install and run `localtunnel` instead of `dev tunnel`.|
|Cloud VM|Develop your project on cloud VM (e.g., [Azure VMs](https://azure.microsoft.com/products/virtual-machines/) or [Azure DevTest Labs](https://azure.microsoft.com/products/devtest-lab/)). You can choose either to still use dev tunnel on your cloud VM, or to directly expose your bot service via VM's public hostname and port.|

**1. Use ngrok and automatically set the tunnel endpoint.**

If you opt to use ngrok, you can install [ngrok](https://ngrok.com/), modify the tunnel task in `.vscode/tasks.json` and add the following script action to your `teamsapp.local.yml` file to obtain the ngrok tunnel endpoint and simplify the debugging process:

**a. Update `Start local tunnel` task:**
```json
{
    "label": "Start local tunnel",
    "type": "shell",
    "command": "ngrok http 3978 --log=stdout --log-format=logfmt",
    "isBackground": true,
    "problemMatcher": {
        "pattern": [
            {
                "regexp": "^.*$",
                "file": 0,
                "location": 1,
                "message": 2
            }
        ],
        "background": {
            "activeOnStart": true,
            "beginsPattern": "starting web service",
            "endsPattern": "started tunnel|failed to reconnect session"
        }
    }
}
```
**b. Add the following action in the first step of the provision lifecycle in `teamsapp.local.yml`:**
- Linux and macOS
  ```yml
  provision:
    - uses: script
      with:
        run: |
          for i in {1..10}; do
            endpoint=$(curl -s localhost:4040/api/tunnels | grep -o 'https://[a-zA-Z0-9 -\.]*\.ngrok\.io')
            if [ -n "$endpoint" ]; then
              break
            fi
            sleep 10
          done
          if [ -z "$endpoint" ]; then
            echo "ERROR: Failed to find tunnel endpoint after 10 attempts."
            exit 1
          else
            echo "::set-teamsfx-env BOT_ENDPOINT=$endpoint"
            echo "::set-teamsfx-env BOT_DOMAIN=${endpoint:8}"
          fi
  ```
- Windows
  ```yml
  provision:
    - uses: script
      with:
        run: |
          for ($i = 1; $i -le 10; $i++) {
            $endpoint = (Invoke-WebRequest -Uri "http://localhost:4040/api/tunnels" | Select-String -Pattern 'https://[a-zA-Z0-9 -\.]*\.ngrok\.io').Matches.Value
            if ($endpoint) {
              break
            }
            sleep 10
          }
          if (-not $endpoint) {
            echo "ERROR: Failed to find tunnel endpoint after 10 attempts."
            exit 1
          } else {
            echo "::set-teamsfx-env BOT_ENDPOINT=$endpoint"
            echo "::set-teamsfx-env BOT_DOMAIN=$($endpoint.Substring(8))"
          }
  ```

**2. Manually update the tunnel endpoint.**

For any tunnel service, you can skip or remove the Teams Toolkit dev tunnel task and manually specify your messaging endpoint by setting it in the botFramework/create action in `teamsapp.local.yml`.

```yml
provision:
  - uses: botFramework/create # Create or update the bot registration on dev.botframework.com
    with:
      botId: ${{BOT_ID}}
      name: bot
      messagingEndpoint: your-messaging-endpoint
      description: ""
      channels:
        - name: msteams
```

```json
{
    "label": "Start Teams App Locally",
    "dependsOn": [
        "Validate prerequisites",
        // Remove/comment out tunnel task
        // "Start local tunnel",
        "Provision",
        "Deploy",
        "Start application"
    ],
    "dependsOrder": "sequence"
}
```

### Provision

This task executes lifecycle *provision* to prepare Teams app related resources required for debugging. It references `teamsapp.local.yml`, so the steps and actions can be customized in `teamsapp.local.yml`.

| Arguments | Type | Required | Description |
|---|---|---|---|
| env | string | required | Environment name. |

#### Sample
```json
{
    "label": "Provision",
    "type": "teamsfx",
    "command": "provision",
    "args": {
        "env": "local"
    }
}
```

### Deploy

This task executes lifecycle *deploy* to build project. It references `teamsapp.local.yml`, so the steps and actions can be customized in `teamsapp.local.yml`.

| Arguments | Type | Required | Description |
|---|---|---|---|
| env | string | required | Environment name. |

#### Sample
```json
{
    "label": "Build project",
    "type": "teamsfx",
    "command": "deploy",
    "args": {
        "env": "local"
    }
}
```

### Start application

These tasks are standard VS Code shell tasks to execute npm commands on project, following the [official schema defined by VS Code](https://code.visualstudio.com/docs/editor/tasks). For example,

- `command` defines the shell command to be executed
- `isBackground: true` means the task keeps running in the background
- `problemMatcher` is used to capture the begin/end or any problem from the task output

#### Sample
```json
{
    "label": "Start application",
    "dependsOn": [
        "Start frontend"
    ]
},
{
    "label": "Start frontend",
    "type": "shell",
    "command": "npm run dev:teamsfx",
    "isBackground": true,
    "options": {
        "cwd": "${workspaceFolder}"
    },
    "problemMatcher": {
        "pattern": {
            "regexp": "^.*$",
            "file": 0,
            "location": 1,
            "message": 2
        },
        "background": {
            "activeOnStart": true,
            "beginsPattern": ".*",
            "endsPattern": "Compiled|Failed"
        }
    }
}
```

### Launch Teams Web Client
This Task is to launch your application in Teams Web Client for remote development environment (e.g. Codespaces).
| Arguments | Type | Required | Description |
|---|---|---|---|
| env | string | required | Environment name. |
| manifestPath | string | required | The file path to the Teams manifest template file. |

#### Sample
```json
{
  "label": "Launch Teams for Codespaces",
  "type": "teamsfx",
  "command": "launch-web-client",
  "args": {
    "env": "local",
    "manifestPath": "${workspaceFolder}/appPackage/manifest.json"
  }
}
```

### Launch desktop client

This task is to launch Teams desktop client.

| Arguments | Type | Required | Description |
|--|--|--|--|
| url            | string | required | The Teams desktop client url with app id. |

#### Sample
```json
{
    "label": "Start desktop client",
    "type": "teamsfx",
    "command": "launch-desktop-client",
    "args": {
        "url": "teams.microsoft.com/l/app/${{local:TEAMS_APP_ID}}?installAppPackage=true"
    }
}
```