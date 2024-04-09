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

# Retrieve data from different sources

## Azure AI Search as Data Source

This doc showcases a solution to:
- Add your document to Azure AI Search via Azure OpenAI Service
- Use Azure AI Search index as data source in the RAG app

### Data Ingestion

With [Azure OpenAI on your data](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/use-your-data?tabs=ai-search), you can ingest your knowledge documents to Azure AI Search Service and create a vector index. Then you can use the index as data source.

- Prepare your data in Azure Blob Storage, or directly upload in later step
- On Azure OpenAI Studio, add your data source
  ![AOAI Data Source](https://github.com/OfficeDev/TeamsFx/assets/13211513/a5ca2e74-b95e-4c02-bc03-e06aec7208a3)
- Fill fields to create a vector index
  ![AOAI Data Source Step](a[ssets/aoai-datasource-step.png](https://github.com/OfficeDev/TeamsFx/assets/13211513/d86106bf-578f-4d9f-8c20-aaf6d02b4d33))

> Note: this approach creates an end-to-end chat API to be called as AI model. But you can also just use the created index as data source, and use Teams AI library to customize the retrieval and prompt.

### Data Source Impelmentation

After ingesting data into Azure AI Search, you can implement your own `DataSource` to retrieve data from search index.

```typescript
import { AzureKeyCredential, SearchClient } from "@azure/search-documents";
import { DataSource, Memory, OpenAIEmbeddings, RenderedPromptSection, Tokenizer } from "@microsoft/teams-ai";
import { TurnContext } from "botbuilder";

export interface Doc {
  id: string,
  content: string, // searchable
  filepath: string,
  // contentVector: number[] // vector field
  // ... other fields
}

// Azure OpenAI configuration
const aoaiEndpoint = "<your-aoai-endpoint>";
const aoaiApiKey = "<your-aoai-key>";
const aoaiDeployment = "<your-embedding-deployment, e.g., text-embedding-ada-002>";

// Azure AI Search configuration
const searchEndpoint = "<your-search-endpoint>";
const searchApiKey = "<your-search-apikey>";
const searchIndexName = "<your-index-name>";

export class MyDataSource implements DataSource {
  public readonly name = "my-datasource";
  private readonly embeddingClient: OpenAIEmbeddings;
  private readonly searchClient: SearchClient<Doc>;

  constructor() {
    this.embeddingClient = new OpenAIEmbeddings({
      azureEndpoint: aoaiEndpoint,
      azureApiKey: aoaiApiKey,
      azureDeployment: aoaiDeployment
    });
    this.searchClient = new SearchClient<Doc>(searchEndpoint, searchIndexName, new AzureKeyCredential(searchApiKey));
  }

  public async renderData(context: TurnContext, memory: Memory, tokenizer: Tokenizer, maxTokens: number): Promise<RenderedPromptSection<string>> {
    // use user input as query
    const input = memory.getValue("temp.input") as string;

    // generate embeddings
    const embeddings = (await this.embeddingClient.createEmbeddings(input)).output[0];

    // query Azure AI Search
    const response = await this.searchClient.search(input, {
      select: [ "id", "content", "filepath" ],
      searchFields: ["rawContent"],
      vectorSearchOptions: {
        queries: [{
          kind: "vector",
          fields: [ "contentVector" ],
          vector: embeddings,
          kNearestNeighborsCount: 3
        }]
      }
      queryType: "semantic",
      top: 3,
      semanticSearchOptions: {
        // your semantic configuration name
        configurationName: "default",
      }
    });

    // Add documents until you run out of tokens
    let length = 0, output = '';
    for await (const result of response.results) {
      // Start a new doc
      let doc = `${result.document.content}\n\n`;
      let docLength = tokenizer.encode(doc).length;
      const remainingTokens = maxTokens - (length + docLength);
      if (remainingTokens <= 0) {
          break;
      }

      // Append do to output
      output += doc;
      length += docLength;
    }
    return { output, length, tooLong: length > maxTokens };
  }
}
```