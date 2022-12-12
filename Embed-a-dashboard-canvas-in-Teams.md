> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

The dashboard tab template from Teams Toolkit enables you to quickly get started with embedding a canvas containing multiple cards that provide an overview of data or content in Microsoft Teams while worrying less about dashboard layout.

## In this tutorial, you will learn

Basic Concepts:
 * [What are Teams tabs](https://learn.microsoft.com/microsoftteams/platform/tabs/what-are-tabs)
 * [App design guidelines for Tab](https://learn.microsoft.com/microsoftteams/platform/tabs/design/tabs)
 * [Fluent UI Library](https://react.fluentui.dev/?path=/docs/concepts-introduction--page) and [Fluent UI React Charting Examples](https://fluentuipr.z22.web.core.windows.net/heads/master/react-charting/demo/index.html#/)

Get started with Teams Toolkit:
 * [How to create a new dashboard tab](#create-a-new-dashboard-project)
 * [How to understand the dashboard tab project](#take-a-tour-of-your-app-source-code)

Customize the scaffolded app template:
 * [How to add a widget](#how-to-add-a-new-widget)
 * [How to add a dashboard](#how-to-add-a-new-dashboard)
 * [How to add a Graph API call](#how-to-add-a-new-graph-api-call)

## Create a new dashboard project

### In Visual Studio Code

1. From Teams Toolkit side bar click `Create a new Teams app` or select `Teams: Create a new Teams app` from the command palette.

![image](https://user-images.githubusercontent.com/11220663/205845061-ef156620-b2c3-4d5c-9ad5-82291a6b0987.png)

2. Select `Create a new Teams app`.

![image](https://user-images.githubusercontent.com/11220663/205845175-b2c1e6cf-5ae9-4a48-a830-dca456f4a8ca.png)

3. Select `Dashboard tab` from Scenario-based Teams app section.

![image](https://user-images.githubusercontent.com/11220663/205845275-40b69e76-6bda-42a5-af7e-0ac74a7357ca.png)

4. Select programming language.

![image](https://user-images.githubusercontent.com/11220663/205845342-ce213cdb-a42a-4f6a-a27d-4a12dc3cb829.png)

5. Select a workspace folder.

![image](https://user-images.githubusercontent.com/11220663/205845434-f44ca178-f481-4674-bb9a-80324e6459df.png)

6. Enter an application name and then press enter.

![image](https://user-images.githubusercontent.com/11220663/205845475-a718966e-8a2b-4300-b454-a569362df67c.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.

* If you prefer non-interactive mode, enter all required parameters in one command. 

`teamsfx new --interactive false --capabilities "dashboard-tab" --programming-language "typescript" --folder "./" --app-name dashboard-cli-001`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. after running this template and you will see a tab app loaded as below:

![image](https://user-images.githubusercontent.com/11220663/205831064-26f94d61-7bb7-4bc1-8678-c52be6cb27b3.png)

This app also supported teams different themes, including dark theme and high contrast theme.

|           Dark theme           |     High contrast theme      |
| :----------------------------: | :--------------------------: |
| <img src="https://user-images.githubusercontent.com/11220663/205836054-52777c73-7598-4f30-9284-62917fab865d.png" width="500" height="300" /> | <img src="https://user-images.githubusercontent.com/11220663/205836124-ffc16d63-0148-4b72-bb70-c304174ef5e7.png" width="500" height="300" /> |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Take a tour of your app source code

This section walks through the generated code. The project folder contains the following:

| Folder      | Contents                                                                          |
| ----------- | --------------------------------------------------------------------------------- |
| `.fx`       | Project level settings, configurations, and environment information               |
| `.vscode`   | VSCode files for local debug                                                      |
| `tabs`      | The source code for the dashboard tab Teams application                           |
| `templates` | Templates for the Teams application manifest and for provisioning Azure resources |

The core dashboard implementation is in `tabs` folder.

The following files provide the business logic for the dashboard tab. These files can be updated to fit your business logic requirements. The default implementation provides a starting point to help you get started.

| File                                       | Contents                                          |
| ------------------------------------------ | ------------------------------------------------- |
| `src/models/listModel.tsx`                 | Data model for the list widget                    |
| `src/services/listService.tsx`             | A data retrive implementation for the list widget |
| `src/views/dashboards/SampleDashboard.tsx` | A sample dashboard layout implementation          |
| `src/views/lib/Dashboard.styles.ts`        | The dashbaord style file                          |
| `src/views/lib/Dashboard.tsx`              | An base class that defines the dashboard          |
| `src/views/lib/Widget.styles.ts`           | The widgt style file                              |
| `src/views/lib/Widget.tsx`                 | An abstract class that defines the widget         |
| `src/views/styles/ListWidget.styles.ts`    | The list widget style file                        |
| `src/views/widgets/ChartWidget.tsx`        | A widget implementation that can display a chart  |
| `src/views/widgets/ListWidget.tsx`         | A widget implementation that can display a list   |

The following files are project-related files. You generally will not need to customize these files.

| File                               | Contents                                         |
| ---------------------------------- | ------------------------------------------------ |
| `src/index.tsx`                    | Application entry point                          |
| `src/App.tsx`                      | Application route                                |
| `src/internal/addNewScopes.ts`     | Implementation of new scopes add                 |
| `src/internal/context.ts`          | TeamsFx Context                                  |
| `src/internal/login.ts`            | Implementation of login                          |
| `src/internal/singletonContext.ts` | Implementation of the TeamsFx instance singleton |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new widget

You can use the following steps to add a new widget to the dashboard:

1. [Step 1: Define a data model](#step-1-define-a-data-model)
2. [Step 2: Create a data retrive service](#step-2-create-a-data-retrive-service)
3. [Step 3: Create a widget file](#step-3-create-a-widget-file)
4. [Step 4: Add the widget to the dashboard](#step-4-add-the-widget-to-the-dashboard)

### Step 1: Define a data model

Define a data model based on the business scenario, and put it in `tabs/src/models` folder. The widget model defined according to the data you want to display in the widget. Here's a sample data model:

```typescript
export interface SampleModel {
  content: string;
}
```

### Step 2: Create a data retrive service

Simply, you can create a service that returns dummy data. We recommend that you put data files in the `tabs/src/data` folder, and put data retrive services in the `tabs/src/services` folder.

Here's a sample json file that contains dummy data:

```json
{
  "content": "Hello world!"
}
```

Here's a dummy data retrive service:

```typescript
import { SampleModel } from "../models/sampleModel";
import SampleData from "../data/SampleData.json";

export const getSampleData = (): SampleModel => SampleData;
```

> Note: You can also implement a service to retrieve data from the backend service or from the Microsoft Graph API.

### Step 3: Create a widget file

Create a widget file in `tabs/src/views/widgets` folder. Extend the [`Widget`](tabs/src/views/lib/Widget.tsx) class. The following table lists the methods that you can override to customize your widget.

| Methods           | Function                                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `getData()`       | This method is used to get the data for the widget. You can implement it to get data from the backend service or from the Microsoft Graph API |
| `headerContent()` | Customize the content of the widget header                                                                                                    |
| `bodyContent()`   | Customize the content of the widget body                                                                                                      |
| `footerContent()` | Customize the content of the widget footer                                                                                                    |

> All methods are optional. If you do not override any method, the default widget layout will be used.

Here's a sample widget implementation:

```tsx
import { Button, Text } from "@fluentui/react-components";
import { Widget } from "../lib/Widget";
import { SampleModel } from "../../models/sampleModel";
import { getSampleData } from "../../services/sampleService";

export class SampleWidget extends Widget<SampleModel> {
  async getData(): Promise<SampleModel> {
    return getSampleData();
  }

  headerContent(): JSX.Element | undefined {
    return <Text>Sample Widget</Text>;
  }

  bodyContent(): JSX.Element | undefined {
    return <div>{this.state.data?.content}</div>;
  }

  footerContent(): JSX.Element | undefined {
    return (
      <Button
        appearance="primary"
        size="medium"
        style={{ width: "fit-content" }}
        onClick={() => {}}
      >
        View Details
      </Button>
    );
  }
}
```

### Step 4: Add the widget to the dashboard

1. Go to `tabs/src/views/dashboards/SampleDashboard.tsx`, if you want create a new dashboard, please refer to [How to add a new dashboard](#how-to-add-a-new-dashboard).
2. Update your `dashboardLayout()` method to add the widget to the dashboard:

```tsx
protected dashboardLayout(): void | JSX.Element {
  return (
    <>
      <ListWidget />
      <ChartWidget />
      <SampleWidget />
    </>
  );
}
```

> Note: If you want put your widget in a column, you can use the [`oneColumn()`](tabs/src/views/lib/Dashboard.styles.ts#L32) method to define the column layout. Here is an example:

```tsx
protected dashboardLayout(): void | JSX.Element {
  return (
    <>
      <ListWidget />
      <div style={oneColumn()}>
        <ChartWidget />
        <SampleWidget />
      </div>
    </>
  );
}
```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new dashboard

You can use the following steps to add a new dashboard layout:

1. [Step 1: Create a dashboard class](#step-1-create-a-dashboard-class)
2. [Step 2: Override methods to customize dashboard layout](#step-2-override-methods-to-customize-dashboard-layout)
3. [Step 3: Add a route for the new dashboard](#step-3-add-a-route-for-the-new-dashboard)
4. [Step 4: Modify manifest to add a new dashboard tab](#step-4-modify-manifest-to-add-a-new-dashboard-tab)

### Step 1: Create a dashboard class

Create a file with the extension `.tsx` for your dashboard in the `tabs/src/views/dashboards` directory. For example, `YourDashboard.tsx`. Then, create a class that extends the [Dashboard](tabs/src/views/lib/Dashboard.tsx) class.

```tsx
export default class YourDashboard extends Dashboard {}
```

### Step 2: Override methods to customize dashboard layout

Dashboard class provides some methods that you can override to customize the dashboard layout. The following table lists the methods that you can override.

| Methods             | Function                                                                          |
| ------------------- | --------------------------------------------------------------------------------- |
| `rowHeights()`      | Customize the height of each row of the dashboard                                 |
| `columnWidths()`    | Customize how many columns the dashboard has at most and the width of each column |
| `dashboardLayout()` | Define widgets layout                                                             |

Here is an example to customize the dashboard layout.

```tsx
export default class YourDashboard extends Dashboard {
  protected rowHeights(): string | undefined {
    return "500px";
  }

  protected columnWidths(): string | undefined {
    return "4fr 6fr";
  }

  protected dashboardLayout(): void | JSX.Element {
    return (
      <>
        <SampleWidget />
        <div style={oneColumn("6fr 4fr")}>
          <SampleWidget />
          <SampleWidget />
        </div>
      </>
    );
  }
}
```

> Note: All methods are optional. If you do not override any method, the default dashboard layout will be used.

### Step 3: Add a route for the new dashboard

Open the `tabs/src/App.tsx` file, and add a route for the new dashboard. Here is an example:

```tsx
import YourDashboard from "./views/dashboards/YourDashboard";

export default function App() {
  ...
  <Route exact path="/yourdashboard" component={YourDashboard} />
  ...
}
```

### Step 4: Modify manifest to add a new dashboard tab

Open the [`templates/appPackage/manifest.template.json`](templates/appPackage/manifest.template.json) file, and add a new dashboard tab under the `staticTabs`. Here is an example:

```json
{
  "name": "Your Dashboard",
  "entityId": "yourdashboard",
  "contentUrl": "{{state.fx-resource-frontend-hosting.endpoint}}{{state.fx-resource-frontend-hosting.indexPath}}/yourdashboard",
  "websiteUrl": "{{state.fx-resource-frontend-hosting.endpoint}}{{state.fx-resource-frontend-hosting.indexPath}}/yourdashboard",
  "scopes": ["personal"]
}
```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to add a new Graph API call

### Add SSO First

Before you add your logic of calling a Graph API, you should enable your dashboard project to use SSO. It is convenient to add SSO related files by using `Teams Toolkit`. Refer to the following 2 steps to add SSO.

1. Step 1: Click `Teams Toolkit` in the side bar > Click `Add features` in `DEVELOPMENT`.

   ![image](https://user-images.githubusercontent.com/11220663/205840715-542f2cfb-bf70-440b-ab8a-8e9139ff3685.png)

2. Step 2: Choose `Single Sign-On` to add.

   ![image](https://user-images.githubusercontent.com/11220663/205840810-5906e6b0-0ff8-4587-8967-9c84dabf2683.png)

3. Step 3: Move `auth-start.html` and `auth-end.html` in `auth/tab/public` folder to `tabs/public/`.
   These two HTML files are used for auth redirects.

4. Step 4: Move `sso` folder under `auth/tab` to `tabs/src/sso/`.

Now you have already added SSO files to your project, and you can call Graph APIs. There are two types of Graph APIs, one will be called from the front-end(most of APIs, use delegated permissions), the other will be called from the back-end(sendActivityNotification, e.g., use application permissions). You can refer to [this tutorial](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-beta) to check permission types of the Graph APIs you want to call.

### From the front-end(use delegated permissions)

If you want to call a Graph API from the front-end tab, you can refer to the following steps.

1. [Step 1: Consent delegated permissions first](#step-1-consent-delegated-permissions-first)
2. [Step 2: Create a graph client by adding the scope related to the Graph API you want to call](#step-2-create-a-graph-client-by-adding-the-scope-related-to-the-graph-api-you-want-to-call)
3. [Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end](#step-3-call-the-graph-api-and-parse-the-response-into-a-certain-model-which-will-be-used-by-front-end)

#### Step 1: Consent delegated permissions first

You can call [`addNewScope(scopes: string[])`](/tabs/src/internal/addNewScopes.ts) to consent the scopes of permissions you want to add. And the consented status will be preserved in a global context [`FxContext`](/tabs/src/internal/singletonContext.ts).

You can refer to [the Graph API V1.0](https://learn.microsoft.com/en-us/graph/api/overview?view=graph-rest-1.0) to get the `scope name of the permission` related to the Graph API you want to call.

#### Step 2: Create a graph client by adding the scope related to the Graph API you want to call

You can refer to the following code snippet:

```ts
let teamsfx: TeamsFx;
teamsfx = FxContext.getInstance().getTeamsFx();
const graphClient: Client = createMicrosoftGraphClient(teamsfx, scope);
```

#### Step 3: Call the Graph API, and parse the response into a certain model, which will be used by front-end

You can refer to the following code snippet:

```ts
try {
  const graphApiResult = await graphClient.api("<GRAPH_API_PATH>").get();
  // Parse the graphApiResult into a Model you defined, used by the front-end.
} catch (e) {}
```

### From the back-end(use application permissions)

If you want to call a Graph API from the back-end, you can refer to the following steps. In this tutorial, we use `sendActivityNotification` API for example.

1. [Step 1: Consent application permissions first](#step-1-consent-application-permissions-first)
2. [Step 2: Add an Azure Function](#step-2-add-an-azure-function)
3. [Step 3: Get the `installation id` of your Dashboard app](#step-3-get-the-installation-id-of-your-dashboard-app)
4. [Step 4: Add your logic in Azure Function](#step-4-add-your-logic-in-azure-function)
5. [Step 5: Edit manifest file](#step-5-edit-manifest-file)
6. [Step 6: Call the Azure Function from the front-end](#step-5-call-the-azure-function-from-the-front-end)

#### Step 1: Consent application permissions first

Go to [Azure portal](https://portal.azure.com/) > Click `Azure Active Directory` > Click `App registrations` in the side bar > Click your Dashboard app > Click `API permissions` in the side bar > Click `+Add a permission` > Choose `Microsoft Graph` > Choose `Application permissions` > Find the permissions you need > Click `Add permissions` button in the bottom > Click `✔Grant admin consent for XXX` and then click `Yes` button to finish the admin consent

#### Step 2: Add an Azure Function

In the VS Code side bar, click `Add features` in `Teams Toolkit` > Choose `Azure functions` > Enter the function name

   ![image](https://user-images.githubusercontent.com/11220663/205843051-38beae3e-9e02-4fde-a24d-c9dfd7d58dbd.png)

#### Step 3: Get the `installation id` of your Dashboard app

Go [Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer), and use the following api path to get a response.

```
https://graph.microsoft.com/v1.0/users/{user-id | user-principal-name}/teamwork/installedApps?$expand=teamsAppDefinition
```

In the response, you should find the information of your Dashboard app, and then record the `id` of it as `installationId`, which will be used in step 4.

   ![image](https://user-images.githubusercontent.com/11220663/205842962-1ddc76b1-b095-461f-881a-8f3d453ca188.png)

#### Step 4: Add your logic in Azure Function

In the `index.ts` under the folder named in step 2, you can add the following code snippet to call `sendActivityNotification`

```ts
try {
  // do sth here, to call activity notification api
  //
  const graphClient_userId: Client = await createMicrosoftGraphClient(teamsfx, [
    "User.Read",
  ]);
  const userIdRes = await graphClient_userId.api("/me").get();
  const userId = userIdRes["id"];
  // get installationId
  const installationId =
    "ZmYxMGY2MjgtYjJjMC00MzRmLTgzZmItNmY3MGZmZWEzNmFkIyMyM2NhNWVlMy1iYWVlLTRiMjItYTA0OC03YjkzZjk0MDRkMTE=";
  let postbody = {
    topic: {
      source: "entityUrl",
      value:
        "https://graph.microsoft.com/v1.0/users/" +
        userId +
        "/teamwork/installedApps/" +
        installationId,
    },
    activityType: "taskCreated",
    previewText: {
      content: "New Task Created",
    },
    templateParameters: [
      {
        name: "taskId",
        value: "12322",
      },
    ],
  };

  let teamsfx_app: TeamsFx;
  teamsfx_app = new TeamsFx(IdentityType.App);
  const graphClient: Client = createMicrosoftGraphClient(teamsfx_app, [
    ".default",
  ]);
  await graphClient
    .api("users/" + userId + "/teamwork/sendActivityNotification")
    .post(postbody);
} catch (e) {
  console.log(e);
}
```

#### Step 5: Edit manifest file

In the [templates\appPackage\manifest.template.json](templates\appPackage\manifest.template.json), you should add the following properties, which are align with properties in `postbody` in Step 4.

```json
"activities": {
  "activityTypes": [
    {
      "type": "taskCreated",
      "description": "Task Created",
      "templateText": "{actor} created task {taskId}"
    }
  ]
}
```

#### Step 6: Call the Azure Function from the front-end

Call the Azure Function from the front-end. You can refer to the following code snippet to call the Azure Function.

```ts
const functionName = process.env.REACT_APP_FUNC_NAME || "myFunc";
async function callFunction(teamsfx?: TeamsFx) {
  if (!teamsfx) {
    throw new Error("TeamsFx SDK is not initialized.");
  }
  try {
    const credential = teamsfx.getCredential();
    const apiBaseUrl = teamsfx.getConfig("apiEndpoint") + "/api/";
    // createApiClient(...) creates an Axios instance which uses BearerTokenAuthProvider to inject token to request header
    const apiClient = createApiClient(
      apiBaseUrl,
      new BearerTokenAuthProvider(
        async () => (await credential.getToken(""))!.token
      )
    );
    const response = await apiClient.get(functionName);
    return response.data;
  } catch (e) {}
}
```

Refer to [this sample](https://github.com/OfficeDev/TeamsFx-Samples/blob/dev/hello-world-tab-with-backend/tabs/src/components/sample/AzureFunctions.tsx) for some helps. And you can read [this doc](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference?tabs=blob) for more details.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>