> The feature is in Preview, we appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

An AI agent in Microsoft Teams is a conversational chatbot that can reason with large language models to interact with users to understand the intention and choose a sequence of actions to take so the chatbot can complete common tasks. Example tasks include querying and summarizing information (e.g., "Do you have any flights to DC tomorrow?"), authoring content based on user intent (e.g., "Rephrase this sentence to be more professional"), or acting on the user’s behalf (e.g., "Please cancel my order"). To build an AI agent that works in Microsoft Teams, you will need:

* [Teams AI Library](https://github.com/microsoft/teams-ai) is the SDK designed specifically for this use case. It's predictive engine that can map intents to actions by leveraging provided prompts and topic filters. You can even chain multiple actions together to make building complex workflows easy. 

* [Microsoft Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) enables you to get started with building AI agents fast through a set of ready-to-use application templates, development lifecycle automations in Visual Studio Code.

* (Optional) [Assistants API](https://platform.openai.com/docs/assistants/overview) from OpenAI. You can also optionally use the Assistants API from OpenAI to simplify the creation of an AI agent. OpenAI as a platform, offers pre-built tools such as [Code Interpreter](https://platform.openai.com/docs/assistants/tools/code-interpreter), [Knowledge Retrieval](https://platform.openai.com/docs/assistants/tools/knowledge-retrieval) and [Function Calling](https://platform.openai.com/docs/assistants/tools/knowledge-retrieval) that drastically simplifies the code you need to write for common scenarios.

![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/103cfa64-897d-4384-8c07-bd239aaab2b8)


## Build AI assistant with Assistants API

This app template is built with OpenAI Assistants API and Teams AI Library's built-in coordination. It uses Code Interpreter tool to solve math problems and also Function Calling tool to get city weather.

> Note: Currently, the Knowledge Retrieval tool is not supported by Teams AI library.

### Customize assistant creation

The file `src/creator.ts` creates a new OpenAI Assistant. You can customize the assistant creation by updating the parameters including instruction, model, tools and functions.

### Add functions

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

## Build AI assistant from scratch

This app template builds an Assistant that is capable of chatting with users and helping users manage tasks.

### Customize prompt augmentation

The SDK provides a functionality to augment the prompt. With augmentation:
- The actions defined in `src/prompts/planner/actions.json` will be inserted into the prompt to let LLM know the available functions. 
- An internal piece of prompt text will be inserted into the prompt to instruct LLM to determine which functions to call based on the available functions. This prompt text orders LLM to generate the response in a structured json format.
- The SDK will validate the LLM response and let LLM correct or refine the response if the response is in wrong format.

In `src/prompts/planner/config.json`, configure `augmentation.augmentation_type`. The options are:
- `sequence`: suitable for tasks that require multiple steps or complex logic.
- `monologue`: suitable for tasks that require natural language understanding and generation, and more flexibility and creativity.

### Add functions

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

### Other common customizations

Refer to [Customize Basic AI chatbot](https://aka.ms/teamsfx-basic-ai-chatbot#customize-basic-ai-chatbot) to see other common customizations with Teams AI library.