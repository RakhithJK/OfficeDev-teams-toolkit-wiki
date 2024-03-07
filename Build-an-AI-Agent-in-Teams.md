> The feature is in Preview, we appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

An AI agent in Microsoft Teams is a conversational chatbot that can reason with large language models to interact with users to understand the intention and choose a sequence of actions to take so the chatbot can complete common tasks. Example tasks include querying and summarizing information (e.g., "Do you have any flights to DC tomorrow?"), authoring content based on user intent (e.g., "Rephrase this sentence to be more professional"), or acting on the user’s behalf (e.g., "Please cancel my order"). To build an AI agent that works in Microsoft Teams, you will need:

* [Teams AI Library](https://github.com/microsoft/teams-ai) is the SDK designed specifically for this use case. It's predictive engine that can map intents to actions by leveraging provided prompts and topic filters. You can even chain multiple actions together to make building complex workflows easy. 

* [Microsoft Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) enables you to get started with building AI agents fast through a set of ready-to-use application templates, development lifecycle automations in Visual Studio Code.

* (Optional) [Assistants API](https://platform.openai.com/docs/assistants/overview) from OpenAI. You can also optionally use the Assistants API from OpenAI to simplify the development effot of creating an AI agent. OpenAI as a platform, offers pre-built tools such as [Code Interpreter](https://platform.openai.com/docs/assistants/tools/code-interpreter), [Knowledge Retrieval](https://platform.openai.com/docs/assistants/tools/knowledge-retrieval) and [Function Calling](https://platform.openai.com/docs/assistants/tools/knowledge-retrieval) that drastically simplifies the code you need to write for common scenarios.

![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/103cfa64-897d-4384-8c07-bd239aaab2b8)

## In this tutorial, you will learn:

Get started with Teams Toolkit, Teams AI Library and Assistants API:
* [Prerequisite](#Prerequisites)
* [How to choose between Build New and Build with Assistants API](#How-to-choose-between-Build-New-and-Build-with-Assistants-API)
* [How to create a new AI Agent](#How-to-create-a-new-AI-Agent)
* [How to understand the AI Agent project](#How-to-understand-the-AI-Agent-project)
* [How Teams AI Library is used to create an AI Agent](#How-Teams-AI-Library-is-used-to-create-an-AI-Agent)

Customize the app template:

Build New:
* [Customize prompt augmentation]()
* [Add functions]()

Build with Assistants API:
* [Customize the Assistant creation]()
* [Add functions]()

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

***

## Prerequisites

Building an AI agent is an advanced scenario, it requires basic understanding of how AI orchestration works, advanced concepts like [Planner](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/PLANNER.md) and [Actions](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/ACTIONS.md) that can leverage LLM to generate a plan on how to accomplish a user ask and mix and match atomic functions registered in AI system so that it can recombine them into a series of steps that complete the goal.

> [!Tip]
> If you are not familiar with those concepts, please start with [Build a Basic AI Chatbot in Teams](https://github.com/OfficeDev/TeamsFx/wiki/Build-a-Basic-AI-Chatbot-in-Teams).

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to choose between Build New and Build with Assistants API
| Comparison |Build New       | Build with Assistants API   |
| - | - | - |
| Cost    |     Only costs for LLM services                      | Costs for LLM services and use tools in Assistants API may incur [additional costs](https://help.openai.com/en/articles/8550641-assistants-api) |
| Dev Effort |      Medium   | Relatively Small | 
|LLM Services| Azure OpenAI or OpenAI | OpenAI Only |
|Example Implementations in Template | This app template is capable of chatting with users and helping users manage tasks. | This app templates uses Code Interpreter tool to solve math problems and also Function Calling tool to get city weather.|
|Flexibilities| Offers maximum flexibilities | Currently, the Knowledge Retrieval tool is not supported by Teams AI library |

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to create a new AI Agent

### In Visual Studio Code

1. From Teams Toolkit side bar click `Create a New App` or select `Teams: Create a New App` from the command palette.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/d68427e6-fce8-49aa-bb56-6b94f086f9f1)

2. Select `Custom Copilot`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/f2dcb612-aa90-4755-9dba-7580304b146d)

3. Select `AI Agent`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/54215c4f-dbb1-4f4d-bd98-f846fd79456b)

4. Select a way to start building the agent
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/5175ce67-8203-4178-b0e3-7ba469a04786)

5. Select a Programming Language
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/27f8c031-6a65-4395-bcba-681409beaa18)

> [!Important]
> If you have selected `Build with Assistants API` in previous, you may only continue with OpenAI as Azure OpenAI service has not provided support for Assistants API yet.

6. Select a service to access LLMs
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/053e62b2-f470-44b0-8105-af7d1eecf1cd)

7. Based on your service selection, you can optionally enter the credentials to access OpenAI or Azure OpenAI. Hit enter to skip.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/b8b0a1b5-4771-48d7-97f6-314d4ab9351e)

8. Select a folder where to create you project. The default one is `${HOME}/TeamsApps/`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/284edd44-724f-4b87-8e82-d972fcb60f8c)

9. Enter an application name and then press enter.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/8e8b8872-fc0d-402d-8b45-f6fbf1743cc0)

> [!IMPORTANT]
> Make sure you have entered the credentials to access Azure OpenAI or OpenAI service. If you have skipped entering those in the project creation dialog, follow the README file in the project to specify them.

After you successfully created the project, you can quickly start local debugging via `F5` in Visual Studio Code. Select `Debug in Test Tool (Preview)` debug option to test the application in Teams App Test Tool. Now you can start chatting with your AI agent.

Build New:
![ai agent new](https://github.com/OfficeDev/TeamsFx/assets/37978464/053218b7-cb17-4db4-9b8a-50ca04c1cb55)

Build with Assistants API:
![ai agent with assistants api](https://github.com/OfficeDev/TeamsFx/assets/15644078/90868166-115b-4394-a0d2-272bd985d0aa)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to understand the AI Agent project
Teams Toolkit generates a standard project that has built-in features to demonstrate how a basic AI chatbot works as well as some configuration files that help automate the development experience.

Below are common files used for both options to build an AI Agent:
| Folder       | Contents                                            |
| - | - |
| `.vscode`    | VSCode files for debugging                          |
| `appPackage` | Templates for the Teams application manifest        |
| `env`        | Environment files                                   |
| `infra`      | Templates for provisioning Azure resources          |
| `src`        | The source code for the application                 |

The following are another set of project specific files Teams Toolkit generate. You can [visit a complete guide on Github](https://github.com/OfficeDev/TeamsFx/wiki/Teams-Toolkit-Visual-Studio-Code-v5-Guide#overview) to understand how Teams Toolkit works.

| File                                 | Contents                                           |
| - | - |
|`teamsapp.yml`|This is the main Teams Toolkit project file. The project file defines two primary things:  Properties and configuration Stage definitions. |
|`teamsapp.local.yml`|This overrides `teamsapp.yml` with actions that enable local execution and debugging.|
|`teamsapp.testtool.yml`|This overrides `teamsapp.yml` with actions that enable local execution and debugging in Teams App Test Tool.|

When starting the two options (`Build New` vs `Build with Assistants API`), the differences are mostly in the `src` folders where we demonstrate an example implementation and allow developers to customize. 

### Build New

| File                                 | Contents                                           |
| - | - |
|`src/index.js`| Sets up the bot app server.|
|`src/adapter.js`| Sets up the bot adapter.|
|`src/config.js`| Defines the environment variables.|
|`src/prompts/planner/skprompt.txt`| Defines the prompt.|
|`src/prompts/planner/config.json`| Configures the prompt.|
|`src/prompts/planner/actions.json`| Defines the actions.|
|`src/app/app.js`| Handles business logics for the AI Assistant Bot.|
|`src/app/messages.js`| Defines the message activity handlers.|
|`src/app/actions.js`| Defines the AI actions.|

### Build with Assistants API

| File                                 | Contents                                           |
| - | - |
|`src/index.js`| Sets up the bot app server.|
|`src/adapter.js`| Sets up the bot adapter.|
|`src/config.js`| Defines the environment variables.|
|`src/creator.js`| One-time tool to create OpenAI Assistant.|
|`src/app/app.js`| Handles business logics for the AI Assistant Bot.|
|`src/app/messages.js`| Defines the message activity handlers.|
|`src/app/actions.js`| Defines the AI actions.|

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How Teams AI Library is used to create an AI Agent

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

***

## Customize the application template

### Customize assistant creation

The file `src/creator.ts` creates a new OpenAI Assistant. You can customize the assistant creation by updating the parameters including instruction, model, tools and functions.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

### Add functions (with Assistants API)

When the assistant returns a function that need to be called along with its arguments, the SDK maps the function to the corresponding action that is registered in advance, then calls the action handler and submits the results to the assistant. You can add your functions by registering the actions into the app.
- In `src/app/actions.ts`, define the action handlers.
    ```ts
    // Define your own function
    export async function myFunction(context: TurnContext, state: TurnState, parameters): Promise<string> {
      // Implement your function logic
      ...
      // Return the result
      return "...";
    }
    ```
- In `src/app/app.ts`, register the actions.
    ```ts
    app.ai.action("myFunction", myFunction);
    ```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

### Customize prompt augmentation

The SDK provides a functionality to augment the prompt. With augmentation:
- The actions defined in `src/prompts/planner/actions.json` will be inserted into the prompt to let LLM know the available functions. 
- An internal piece of prompt text will be inserted into the prompt to instruct LLM to determine which functions to call based on the available functions. This prompt text orders LLM to generate the response in a structured json format.
- The SDK will validate the LLM response and let LLM correct or refine the response if the response is in wrong format.

In `src/prompts/planner/config.json`, configure `augmentation.augmentation_type`. The options are:
- `sequence`: suitable for tasks that require multiple steps or complex logic.
- `monologue`: suitable for tasks that require natural language understanding and generation, and more flexibility and creativity.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

### Add functions (Build New)

- In `src/prompts/planner/actions.json`, define your actions schema.
  ```json
  [
      ...
      {
          "name": "myFunction",
          "description": "The function description",
          "parameters": {
              "type": "object",
              "properties": {
                  "parameter1": {
                      "type": "string",
                      "description": "The parameter1 description"
                  },
              },
              "required": ["parameter1"]
          }
      }
  ]
  ```
- In `src/app/actions.ts`, define the action handlers.
    ```ts
    // Define your own function
    export async function myFunction(context: TurnContext, state: TurnState, parameters): Promise<string> {
      // Implement your function logic
      ...
      // Return the result
      return "...";
    }
    ```
- In `src/app/app.ts`, register the actions.
    ```ts
    app.ai.action("myFunction", myFunction);
    ```

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>