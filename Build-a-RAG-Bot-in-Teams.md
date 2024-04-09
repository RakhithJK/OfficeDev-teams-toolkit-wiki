# RAG (Retrieval Augmented Generation)

RAG (Retrieval Augmented Generation) is a framework that can incorporate real-time, dynamic, and specified external data sources into Large Language Model (LLM), to generate up to date and contextually accurate responses.

For example:
- **Knowledge base** - "Company's shuttle bus may be 15 minutes late on rainy days."
- **User ask** - "When will the shuttle bus arrive?"
- **AI response with RAG** - "Today is rainy, the shuttle bus may be 15 minutes late than usual, so around 9:15 AM."

## How RAG works with teams-ai library?

A typical RAG architecture has two main flows:

- **Data Ingestion** - an one-time/regular/standalone process, that external knowledge bases are ingested into some data storages, to be queried or searched later.
- **Retrieval and Generation** - a real-time process, that on every user input, retrieve relavant data source from storage, inject into a prompt, and let AI to summarize or generate response.

![RAG typical architecture](https://github.com/OfficeDev/TeamsFx/assets/13211513/30f81050-9db4-4680-aa1d-9db53df1ecaf)

### How teams-ai helps Data Ingestion

In AI context, vector databases are widely used as RAG storages, which store embeddings data and provide vector-similarity search.

Teams-AI library provides utilities to help create embeddings for the given inputs.
```typescript
// create OpenAIEmbeddings instance
const model = new OpenAIEmbeddings({ ... endpoint, apikey, model, ... });

// create embeddings for the given inputs
const embeddings = await model.createEmbeddings(model, inputs);

// your own logic to process embeddings
```

> Note: Teams-AI library does not provide vector database implementation, so you need to add your own logic for further processing the created embeddings.

### How teams-ai helps Retrieval and Generation

Teams-AI library provides functionalities to ease each step of the retrieval and generation process.

![Teams AI helps RAG](https://github.com/OfficeDev/TeamsFx/assets/13211513/7b1d14b1-ac05-4b2e-b8f6-7f2f8aab5e8f)

**Handle input**

The most straightforward way is to pass user's input as is to retrieval. However, if you'd like to customize the input before retrieval, you can [add activity handler](./AddActivityHandlers.md) to certain incoming activities.

**Retrieve datasource**

Teams-AI library provides `DataSource` interface to let you to add your own retrieval logic. You will need to create your own `DataSource` instance, and the library orchestrator will call it on demand.
    
```typescript
class MyDataSource implements DataSource {
  /**
   * Name of the data source.
   */
  public readonly name = "my-datasource";

  /**
   * Renders the data source as a string of text.
   * @param context Turn context for the current turn of conversation with the user.
   * @param memory An interface for accessing state values.
   * @param tokenizer Tokenizer to use when rendering the data source.
   * @param maxTokens Maximum number of tokens allowed to be rendered.
   * @returns The text to inject into the prompt as a `RenderedPromptSection` object.
   */
  renderData(
    context: TurnContext,
    memory: Memory,
    tokenizer: Tokenizer,
    maxTokens: number
  ): Promise<RenderedPromptSection<string>> {
    ...
  }
}
```

**Call AI with prompt**

In Teams-AI's prompt system, you can easily inject data source by adjusting the `augmentation.data_sources` configuration section, e.g., in prompt's `config.json` file:

```json
{
    "schema": 1.1,
    ...
    "augmentation": {
        "data_sources": {
            "my-datasource": 1200
        }
    }
}
```

This connects the prompt with the added `DataSource` in previous step, and library orchestrator will inject the data source text into final prompt. See [AuthorPrompt](./AuthorPrompt.md) for the details.

**Build response**

By default, Teams-AI library replies the AI generated response as text message to user. If you'd like to customize the response, you can override the default SAY action (see [AI Actions](./AddAIActions.md)) or explicitly call AI model (see [AI Models](./AddAIModels.md)) to build your own replies, e.g., with adaptive cards.

## Quick Start

Here's a minimal set of implementations to add RAG to your app. In general, it implements `DataSource` to inject your own knowledge into prompt, so that AI can generate response based on the knowledge.

- Create **myDataSource.ts** to implement `DataSource` interface.
  ```typescript
  export class MyDataSource implements DataSource {
    public readonly name = "my-datasource";
    public async renderData(
      context: TurnContext,
      memory: Memory,
      tokenizer: Tokenizer,
      maxTokens: number
    ): Promise<RenderedPromptSection<string>> {
      const input = memory.getValue('temp.input') as string;
      let knowledge = "There's no knowledge found.";

      // hard-code knowledge
      if (input?.includes("shuttle bus")) {
        knowledge = "Company's shuttle bus may be 15 minutes late on rainy days.";
      } else if (input?.includes("cafe")) {
        knowledge = "The Cafe's available time is 9:00 to 17:00 on working days and 10:00 to 16:00 on weekends and holidays."
      }
      
      return {
        output: knowledge,
        length: knowledge.length,
        tooLong: false
      }
    }
  }
  ```

- In **app.ts**, register the data source.
  ```typescript
  // Register your data source to prompt manager
  planner.prompts.addDataSource(new MyDataSource());
  ```

- Create **prompts/qa/skprompt.txt** for prompt template text.
  ```text
  The following is a conversation with an AI assistant. The assistant is helpful, creative, clever, and very friendly to answer user's question.

  Base your answer off the text below:
  ```

- Create **prompts/qa/config.json** to connect with the data source.
  ```json
  {
      "schema": 1.1,
      "description": "Chat with QA Assistant",
      "type": "completion",
      "completion": {
          "model": "gpt-35-turbo",
          "completion_type": "chat",
          "include_history": true,
          "include_input": true,
          "max_input_tokens": 2800,
          "max_tokens": 1000,
          "temperature": 0.9,
          "top_p": 0.0,
          "presence_penalty": 0.6,
          "frequency_penalty": 0.0,
          "stop_sequences": []
      },
      "augmentation": {
          "data_sources": {
              "my-datasource": 1200
          }
      }
  }
  ```

### Retrieve data from different sources

In real scenario, you may have your knowledge stored somewhere else.

[Azure AI Search as Data Source](./RAG-AzureAISearch.md) provides a sample to add your documents to Azure AI Search Service, then use the search index as data source.

[Microsoft Graph Search API as Data Source](./RAG-MicrosoftGraph.md) provides a sample to use M365 content from Microsoft Graph Search API as data source.

Or, to fully control the data ingestion, see the sample on [Build your own Data Ingestion](./RAG-BuildYourDataIngestion.md) to build your own vector index, and use it as data source.

There are other alternatives, e.g., Azure Cosmos DB Vector Database Extension or Azure PostgreSQL Server pgvector Extension as vector databases, or Bing Web Search API to get latest web content. You may implement any `DataSource` instance to connect with your own data source.

### Pre-process user input

In real scenario, you may want to pre-process user input before retrieval, e.g., to shorten and summarize long question, or to remove sensitive information, or to add some context to the input.

[Pre-process User Input](./RAG-PreProcessUserInput.md) provides a sample to summarize user input to some keywords before retrieval.