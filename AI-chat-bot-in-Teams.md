# AI chat bot in Teams

Microsoft Teams Toolkit enables you to build an AI chatbot that responds to user questions like ChatGPT. This enables your users to talk with the AI bot in Teams.

The app template is built using Teams AI library, which provides the capabilities to integrate with large language models (LLMs).

## Customize AI chatbot

You can add customizations on top of this basic application to build more complex scenarios.

### Customize prompt text

In `src/prompts/chat/skprompt.txt`, author your prompt text. The content written in this file will be inserted into the prompt to instruct LLM. The SDK defines the following syntax that you can use in the prompt text.

**{{$[scope].property}}**: Renders the value of the scoped property that is defined in turn state. The SDK defines three scopes, temp, user and conversation. If scope is omitted, the temp scope will be used.
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

**{{functionName}}**: Calls the specified function and renders the result.
- In `src/app/app.ts`, register the function into prompt manager.
    ```ts
    prompts.addFunction("getTasks", async (context: TurnContext, memory: Memory, functions: PromptFunctions, tokenizer: Tokenizer, args: string[]) => {
      return ...
    });
    ```
- In `src/prompts/chat/skprompt.txt`, use the funtion such as `{{getTasks}}`.

**{{functionName arg1 arg2}}**: Calls the specified function with the provided arguments and renders the result.
- In `src/app/app.ts`, register the function into prompt manager.
- In `src/prompts/chat/skprompt.txt`, use the funtion such as `{{getTasks taskTitle}}`.

### Customize user input

**Whether to include user input**: In `src/prompts/chat/config.json`, configure `completion.include_input`. If `true`, the user input will be appended into the prompt.

**Maximum number of user input tokens**: In `src/prompts/chat/config.json`, configure `completion.max_input_tokens`.

### Customize conversation history

The SDK automatically manages the conversation history and you can customize the followings.

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

**DALL·E**

Not supported currently.

**Whisper**

Not supported currently.

**TTS**

Not supported currently.

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
