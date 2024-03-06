# Basic AI chatbot in Teams

> The feature is in Preview, we appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

[Teams AI Library](https://github.com/microsoft/teams-ai) is a SDK that specifically designed to assist you in creating bots capable of interacting with Teams and Microsoft 365 applications. It is constructed using the [Bot Framework SDK](https://github.com/microsoft/botbuilder-js) as its foundation, simplifying the process of developing bots that interact with Teams' artificial intelligence capabilities.

Microsoft Teams Toolkit is an extension in Visual Studio Code that enables you to get started with building a basic AI chatbot in Temas with ready-to-use application templates written in JavaScript, TypeScript and Python and task automations. 

![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/a2a04a5d-c7a8-479a-a2ae-506a6fed4d60)

## In this tutorial, you will learn:

Get started with Teams Toolkit and Teams AI Library
* [How to create a new basic ai chatbot]()
* [How to understand the basic ai chatbot project]()
* [How Teams AI Chatbot works]()

Customize the app template
* [How to customize the prompts]()
* [How to customize the user inputs]()
* [How to customize conversation history]()
* [How to customize model type]()
* [How to customize completion type]()
* [How to customize the model parameters]()
* [How to handle messages with image]()

***

## Create a new basic ai chatbot project

### In Visual Studio Code

1. From Teams Toolkit side bar click `Create a New App` or select `Teams: Create a New App` from the command palette.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/d68427e6-fce8-49aa-bb56-6b94f086f9f1)

2. Select `Custom Copilot`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/f2dcb612-aa90-4755-9dba-7580304b146d)

3. Select `Basic AI Chatbot`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/033e6a76-f939-4cd3-9faf-c1f66169e31a)

4. Select a Programming Language
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/27f8c031-6a65-4395-bcba-681409beaa18)

5. Select a service to access LLMs
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/053e62b2-f470-44b0-8105-af7d1eecf1cd)

6. Based on your service selection, you can optionally enter the credentials to access OpenAI or Azure OpenAI. Hit enter to skip.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/b8b0a1b5-4771-48d7-97f6-314d4ab9351e)

7. Select a folder where to create you project. The default one is `${HOME}/TeamsApps/`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/284edd44-724f-4b87-8e82-d972fcb60f8c)

8. Enter an application name and then press enter.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/8e8b8872-fc0d-402d-8b45-f6fbf1743cc0)

> [!IMPORTANT]
> Make sure you have entered the credentials to access Azure OpenAI or OpenAI service. If you have skipped entering those in the project creation dialog, follow the README file in the project to specify them.

After you successfully created the project, you can quickly start local debugging via `F5` in Visual Studio Code. Select `Debug in Test Tool (Preview)` debug option to test the application in Teams App Test Tool. Now you can start chatting with your AI-powered bot

![ai chat bot](https://github.com/OfficeDev/TeamsFx/assets/9698542/9bd22201-8fda-4252-a0b3-79531c963e5e)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Understand the Basic AI Chatbot project
Teams Toolkit generates a standard project that has built-in features to demonstarte how a basic AI chatbot works as well as some configuration files that helps automate the development experience.

| Folder       | Contents                                            |
| - | - |
| `.vscode`    | VSCode files for debugging                          |
| `appPackage` | Templates for the Teams application manifest        |
| `env`        | Environment files                                   |
| `infra`      | Templates for provisioning Azure resources          |
| `src`        | The source code for the application                 |

The following files can be customized and demonstrate an example implementation to get you started.

| File                                 | Contents                                           |
| - | - |
|`src/index.js`| Sets up the bot app server.|
|`src/adapter.js`| Sets up the bot adapter.|
|`src/config.js`| Defines the environment variables.|
|`src/prompts/chat/skprompt.txt`| Defines the prompt.|
|`src/prompts/chat/config.json`| Configures the prompt.|
|`src/app/app.js`| Handles business logics for the Basic AI Chatbot.|

The following are Teams Toolkit specific project files. You can [visit a complete guide on Github](https://github.com/OfficeDev/TeamsFx/wiki/Teams-Toolkit-Visual-Studio-Code-v5-Guide#overview) to understand how Teams Toolkit works.

| File                                 | Contents                                           |
| - | - |
|`teamsapp.yml`|This is the main Teams Toolkit project file. The project file defines two primary things:  Properties and configuration Stage definitions. |
|`teamsapp.local.yml`|This overrides `teamsapp.yml` with actions that enable local execution and debugging.|
|`teamsapp.testtool.yml`|This overrides `teamsapp.yml` with actions that enable local execution and debugging in Teams App Test Tool.|

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How Teams AI Chatbot works

Teams-AI library provides a typical flow to build an intelligent chatbot with AI capabilities.

![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/dd22e32b-5087-485d-8e47-3433e66ec76d)

> [!TIP]
> Click here to learn more about [Turn Context and Turn States](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/TURNS.md).

### Turn Context and Turn State
At the beginning to handle any incoming requests to your bot, Teams AI library prepares `TurnContext` and `TurnState` objects. Those are the key objects going through the entire messaging process flow.
* `TurnContext`: The turn context object provides information about the activity such as the sender and receiver, the channel, and other data needed to process the activity.
* `TurnState`: The turn state object stores cookie-like data for the current turn. Just like the turn context, it is carried through the entire application logic, including the activity handlers and the AI System. 

### Pre Processing
After loading the turn state, Teams AI library executes a `before-turn-handler`. This is built on top of the Microsoft Bot Framework that allows you to modify the `TurnState`, but in general you don't need to customize it.

> [!Important]
> On each turn, Teams AI library sequentially goes through all registered handlers to see if the incoming activity matches the selector. Thus, only the first matched handler will be executed.

### Activity Handlers
After that, Teams AI library executes a set of registered activity handlers. This enables developers to handle several types of activities. The activity handler system is the primary way to implement bot or message extension application logic. It is a set of methods and configurations that allows you to register callbacks (known as route handlers), which will trigger based on the incoming activity. These can be in the form of a message, message reaction, or virtually any interaction within the Teams app.

The Teams AI library provides a set of handler registration APIs, e.g., `app.message(text, handler)` to handle specific text message input. Below is a sequential diagram shows how a bot handles the incoming activity on each turn:
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/3ce415b6-f7be-49f7-8c91-3bad6dc35598)

Aother typical usage of message handler is to decorate the messages sent by your bot using rich UI elements such as Adaptive Cards, you can design and iterate over your cards with [Microsoft Adaptive Card Previewer](https://learn.microsoft.com/microsoftteams/platform/concepts/build-and-test/adaptive-card-previewer?tabs=codelens).

> [!Tip]
> Check [Teams AI Library concept documentations](https://github.com/microsoft/teams-ai/tree/main/getting-started/CONCEPTS) here to learn more about other important concepts for the AI System such as Action Planner, Prompt Management and Retrieval Augmented Generations (RAG).

### The AI System
The AI system in Teams AI library is responsible for moderating input and output, generating plans, and executing them. It can be used free standing or routed to by the Application object. Those are the most important concepts you need to know:
* [Prompt Manager](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/PROMPTS.md): Prompts play a crucial role in communicating and directing the behavior of Large Language Models (LLMs) AI. 
* [Planner](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/PLANNER.md): The planner receives the user's ask and returns a plan on how to accomplish the request. The user's ask is in the form of a prompt or prompt template. It does this by using AI to mix and match atomic functions (called actions) registered to the AI system so that it can recombine them into a series of steps that complete a goal.
* [Actions](https://github.com/microsoft/teams-ai/blob/main/getting-started/CONCEPTS/ACTIONS.md): An action is an atomic function that is registered to the AI System. It is a fundamental building block of a plan.

### Post Processing and respond to user
If there is activity handler matched and executed, it goes into after-turn-handler, which enables developers to customize the post-processing.

After post-processing, Teams AI library saves the state and bot can send the response to user.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Customize Basic AI Chatbot

You can add customizations on top of this basic application to build more complex scenarios.

### Customize prompt
Prompts play a crucial role in communicating and directing the behavior of Large Language Models (LLMs) AI. They serve as inputs or queries that users can provide to elicit specific responses from a model. Here's a prompt that asks the LLM for name suggestions:

_Input:_

```
Give me 3 name suggestions for my pet golden retriever.
```

_Response:_

```
Some possible name suggestions for a pet golden retriever are:

- Bailey
- Sunny
- Cooper
```

Using project generated with Teams Toolkit, you can author the prompts in `src/prompts/chat/skprompt.txt` file. The prompts written in this file will be inserted into the prompt used to instruct the LLM. Teams AI library defines the following syntax that you can use in the prompt text.

#### Syntax 1: `{{ $[scope].property }}`
`{{ $[scope].property }}` Renders the value of the scoped property that is defined in turn state. Teams AI library defines three scopes: `temp`, `user` and `conversation`. If scope is omitted, the `temp` scope will be used.

The `{{$[scope].property}}` is used in the following way:
- In `src/app/turnState.ts`, define your temp state, user state, conversation state and application turn state.
    ```ts
    export interface TempState extends DefaultTempState { ... }
    export interface UserState extends DefaultUserState { ... }
    export interface ConversationState extends DefaultConversationState {
        tasks: Record<string, Task>;
    }
    export type ApplicationTurnState = TurnState<ConversationState, UserState, TempState>;
    ``` 
- In `src/app/app.ts`, use application turn state to initialize application.
    ```ts
    const app = new Application<ApplicationTurnState>(...);
    ```
- In `src/prompts/chat/skprompt.txt`, use the scoped state property such as `{{$conversation.tasks}}`.

#### Syntax 2: `{{ functionName }}`
To call an external function and embed the result in your text, use the `{{ functionName }}` syntax. For example, if you have a function called `getTasks` that can return a list of task items, you can embed the results into the responses from LLM:

1. Register the function into prompt manager in `src/app/app.ts`:

    ```ts
    prompts.addFunction("getTasks", async (context: TurnContext, memory: Memory, functions: PromptFunctions, tokenizer: Tokenizer, args: string[]) => {
      return ...
    });
    ```
2. Use the fucntion in `src/prompts/chat/skprompt.txt`: `Your tasks are: {{ getTasks }}`.

#### Syntax 3: ` {{ functionName arg1 arg2 }}`
This syntax enables you to call the specified function with the provided arguments and renders the result. Similar to the usage of calling a function, you can:

1. Register the function into prompt manager in `src/app/app.ts`.
2. Use the function in `src/prompts/chat/skprompt.txt` such as `Your task is: {{ getTasks taskTitle }}`.

### Customize user input

Teams AI library allows you to augment the prompt sent to LLM by including the user inputs. When including user inputs, you need to specify it in a prompt configuration file by setting `completion.include_input` to `true` in `src/prompts/chat/config.json`. You can alos optionally configure the maximum number of user input tokens in `src/prompts/chat/config.json` by changing `completion.max_input_tokens`. The default token is 2048.

> [!Important]
> Note that the configuration properties in the file do not include all the possible configurations. To learn more about the description of each configuration and all the supported configurations see the [PromptTemplatConfig](https://github.com/microsoft/teams-ai/blob/2d43f5ca5b3bf27844f760663641741cae4a3243/js/packages/teams-ai/src/prompts/PromptTemplate.ts#L46C18-L46C39) Typescript interface.

### Customize conversation history

The SDK automatically manages the conversation history, and you can customize the following.

**Whether to include history.** In `src/prompts/chat/config.json`, configure `completion.include_history`. If `true`, the history will be inserted into the prompt to let LLM aware of the conversation history.

**Maximum number of history messages.** Configure `max_history_messages` when initializing `PromptManager`.
```ts
const prompts = new PromptManager({
  promptsFolder: path.join(__dirname, "../prompts"),
  max_history_messages: 3,
});
```

**Maximum number of history tokens.** Configure `max_conversation_history_tokens` when initializing `PromptManager`.
```ts
  const prompts = new PromptManager({
    promptsFolder: path.join(__dirname, "../prompts"),
    max_conversation_history_tokens: 1000,
});
```

### Customize model type

In `src/prompts/chat/config.json`, configure `completion.model`. Below lists the models whether the SDK supports.

**GPT-3.5**

| Model | Supported |
|----------|------|
| gpt-3.5-turbo | Supported |
| gpt-3.5-turbo-16k | Supported |
| gpt-3.5-turbo-instruct| Not supported from `1.1.0` |

**GPT-4**

| Model | Supported |
|----------|------|
| gpt-4 | Supported |
| gpt-4-32k | Supported |
| gpt-4-vision | Supported |
| gpt-4-turbo | Supported |

**DALL·E**: Not supported currently.
**Whisper**: Not supported currently.
**TTS**: Not supported currently.

### Customize completion type

In `src/prompts/chat/config.json`, configure `completion.completion_type`. Supported options are `chat` and `text`.

### Customize model parameters

In `src/prompts/chat/config.json`, configure the model parameters under `completion`:
- `max_tokens`
- `temperature`
- `top_p`
- `presence_penalty`
- `frequency_penalty`
- `stop_sequences`

### Handle messages with image

- In `src/app/app.ts`, initialize `TeamsAttachmentDownloader`.
    ```ts
    const downloader = new TeamsAttachmentDownloader({
      botAppId: config.botId,
      adapter,
    });
    ```
- In `src/app/app.ts`, use `downloader` to initialize application.
    ```ts
    const app = new Application({
      ...
      fileDownloaders: [downloader]
    });
    ```
- In `src/prompts/chat/config.json`, set `completion.include_images` to `true`. Configure `completion.model` with your model that can handle messages with image.
- In `src/prompts/chat/skprompt.txt`, author your prompt text to handle messages with image.