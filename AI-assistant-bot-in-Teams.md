# AI assistant bot in Teams

Microsoft Teams Toolkit enables you to build applications that act as AI ssistants who can recognize users intents and completing common tasks on behalf of users. Example tasks include querying and summarizing information (e.g., "Do you have any flights to DC tomorrow?"), authoring content based on user intent (e.g., "Rephrase this sentence to be more professional"), or acting on the user’s behalf (e.g., "Please cancel my order"). 

AI ssistants complete common task by using some pre-defined skills. The commonly used skills are:
- **Role Prompting via Chat Completions** to generate a model response for given chat conversation.
- **Invoke API Call via Function Calling**. This is a comprehensive approach to create skills that powered by an API, be it real-time information retrieval or any task completion. After you define a set of functions, the model can convert natural language to arguments so that you can call your internal API.
- **Code Interpreter** is OpenAI hosted tool. This enables developers to run code iteratively to solve challenging code and math problems.
- **Augmented Knowledge** through proprietary product information or documents. This is a scenario that should be coevered separately in Chat with Your Data.

Microsoft Teams Toolkit provides two app templates to build AI assistant bot, both built using Teams AI library, which provides the capabilities to integrate with large language models (LLMs).
- **Start with Assistant API**: This allows you to use existing tools in Open AI platform to build skills for an Assistant. Available tools such as Code Interpreter, Knowledge Retrieval and Function Calling.
- **Start from Scratch**: This requires you to build custom skills for your assistant bot, through basic prompting or function calling using Teams AI library.

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