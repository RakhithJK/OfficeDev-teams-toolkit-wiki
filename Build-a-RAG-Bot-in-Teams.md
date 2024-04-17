> The feature is in Preview, we appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

One of the most powerful applications enabled by LLMs is sophisticated question-answering (Q&A) chatbots. These are applications that can answer questions about specific source information. These applications use a technique known as [Retrieval Augmented Generation](https://python.langchain.com/docs/use_cases/question_answering/#what-is-rag), or RAG. For example:
- **Knowledge Base**: "Company's shuttle bus may be 15 minutes late on rainy days."
- **User Query**: "When will the shuttle bus arrive?"
- **AI Response (With RAG)**: "Today is rainy, the shuttle bus may be 15 minutes late than usual, so around 9:15 AM."

![RAG typical architecture](https://github.com/OfficeDev/TeamsFx/assets/13211513/30f81050-9db4-4680-aa1d-9db53df1ecaf)

This chart has demonstrated a typical RAG architecture that has two main flows:
- **Data Ingestion** - A pipeline for ingesting data from a source and indexing it. This usually happens offline.
- **Retrieval and Generation** - The actual RAG chain, which takes the user query at run time and retrieves the relevant data from the index, then passes that to the model.

Microsoft Teams enables developers to build a conversational bot with RAG capability to create a powerful experience to maximize the productivity.

Teams Toolkit provides a series ready to use application templates under the category `Chat with your data` that combines the capabilities of Azure AI Search, Microsoft 365 & SharePoint and Custom API as different data source and Large Language Models (LLMs) to create a conversational search experience in Microsoft Teams.

## In this tutorial, you will learn:
Get started with Teams Toolkit and Teams AI Library:
* [How to create a new RAG bot]()
* [How to understand the RAG project]()
* [How teams-ai helps to achieve RAG scenario]()

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How to create a new RAG bot
> [!IMPORTANT]
> This flow is based on the choice of `Customize` as your data source, if you choose to start with `Azure AI Service`, `Custom API` or `Microsoft 365` you may see different prompts in Visual Studio Code. Make sure to follow the prompts and instructions in Teams Toolkit.

1. From Teams Toolkit side bar click `Create a New App` or select `Teams: Create a New App` from the command palette.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/115f5f83-0cb1-457f-86ce-f1d1136e840e)

2. Select `Custom Copilot`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/fd3c90a2-6d36-4520-a7e6-ae886b6a0ff5)

3. Select `Chat With Your Data`
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/9416dd1d-88bc-429c-899e-fc20eae3bf51)

4. Select an option for your data source
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/d1a4f909-ab0c-4e54-9b68-5ddc5d7cc526)

5. Select a Programming Language
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/88320899-f28d-4f69-b7e8-9e6f026fa9b7)

6. Select a service to access LLMs
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/18d56ab2-1f70-4bce-ab16-a225e489b9ad)

7. Based on your service selection, you can optionally enter the credentials to access OpenAI or Azure OpenAI. Hit enter to skip.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/096ec112-f9c1-4937-9d2d-33cb54da90ad)

8. Select a folder where to create you project. The default one is `${HOME}/TeamsApps/`.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/4db42284-2eed-487b-bfb8-de92710a4778)

9. Enter an application name and then press enter.
![image](https://github.com/OfficeDev/TeamsFx/assets/11220663/2b183451-8d90-467b-8285-b7ba4c0d757e)

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## How teams-ai helps to achieve RAG scenario

> [!TIP] 
> Teams-AI library does not provide vector database implementation, so you need to add your own logic for further processing the created embeddings.

In AI context, vector databases are widely used as RAG storages, which store embeddings data and provide vector-similarity search. Teams-AI library provides utilities to help create embeddings for the given inputs.

```typescript
// create OpenAIEmbeddings instance
const model = new OpenAIEmbeddings({ ... endpoint, apikey, model, ... });

// create embeddings for the given inputs
const embeddings = await model.createEmbeddings(model, inputs);

// your own logic to process embeddings
```
![Teams AI helps RAG](https://github.com/OfficeDev/TeamsFx/assets/13211513/7b1d14b1-ac05-4b2e-b8f6-7f2f8aab5e8f)
Teams-AI library also provides functionalities to ease each step of the retrieval and generation process.
- **Step 1: Handle Input**: The most straightforward way is to pass user's input as is to retrieval. However, if you'd like to customize the input before retrieval, you can [add activity handler](./AddActivityHandlers.md) to certain incoming activities.

- **Retrieve data source**: Teams-AI library provides `DataSource` interface to let you to add your own retrieval logic. You will need to create your own `DataSource` instance, and the library orchestrator will call it on demand.
    
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

- **Call AI with prompt**: In Teams-AI's prompt system, you can easily inject data source by adjusting the `augmentation.data_sources` configuration section. This connects the prompt with the added `DataSource` in previous step, and library orchestrator will inject the data source text into final prompt. See [AuthorPrompt](./AuthorPrompt.md) for the details. For example, in prompt's `config.json` file:

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

- **Build response**: By default, Teams-AI library replies the AI generated response as text message to user. If you'd like to customize the response, you can override the default SAY action (see [AI Actions](./AddAIActions.md)) or explicitly call AI model (see [AI Models](./AddAIModels.md)) to build your own replies, e.g., with adaptive cards.


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

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Choose Between Data Sources
In the `Chat With Your Data` or RAG scenarios, Teams Toolkit has provided four different types of data source.

[Azure AI Search as Data Source](#azure-ai-search-as-data-source) provides a sample to add your documents to Azure AI Search Service, then use the search index as data source.

[Microsoft Graph Search API as Data Source](#microsoft-graph-search-api-as-data-source) provides a sample to use M365 content from Microsoft Graph Search API as data source.

Or, to fully control the data ingestion, see the sample on [Build your own Data Ingestion](#build-your-own-data-ingestion) to build your own vector index, and use it as data source.

There are other alternatives, e.g., Azure Cosmos DB Vector Database Extension or Azure PostgreSQL Server pgvector Extension as vector databases, or Bing Web Search API to get latest web content. You may implement any `DataSource` instance to connect with your own data source.

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
  ![AOAI Data Source Step](https://github.com/OfficeDev/TeamsFx/assets/13211513/d86106bf-578f-4d9f-8c20-aaf6d02b4d33)

> Note: this approach creates an end-to-end chat API to be called as AI model. But you can also just use the created index as data source, and use Teams AI library to customize the retrieval and prompt.

### Data Source Implementation

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

## Microsoft Graph Search API as Data Source

This doc showcases a solution to query M365 content from Microsoft Graph Search API as data source in the RAG app. To learn more about Microsoft Graph Search API, you can refer to [Use the Microsoft Search API to search OneDrive and SharePoint content](https://learn.microsoft.com/en-us/graph/search-concept-files).
 
**Prerequisite** - You should create a Graph API client and grant it the Files.Read.All permission scope to access SharePoint and OneDrive files, folders, pages, and news.

### Data Ingestion

Microsoft Graph Search API is available for searching SharePoint content, thus you just need to ensure your document is uploaded to SharePoint / OneDrive, no extra data ingestion required.

> Note: SharePoint Server indexes a file only if its file extension is listed on the Manage File Types page. For a complete list of supported file extensions, refer to the [Default crawled file name extensions and parsed file types in SharePoint Server and SharePoint in Microsoft 365](https://learn.microsoft.com/sharepoint/technical-reference/default-crawled-file-name-extensions-and-parsed-file-types).

### Data Source Implementation

The following is an example of search `txt` files in SharePoint and OneDrive.

```typescript
import {
  DataSource,
  Memory,
  RenderedPromptSection,
  Tokenizer,
} from "@microsoft/teams-ai";
import { TurnContext } from "botbuilder";
import { Client, ResponseType } from "@microsoft/microsoft-graph-client";

export class GraphApiSearchDataSource implements DataSource {
  public readonly name = "my-datasource";
  public readonly description =
    "Searches the graph for documents related to the input";
  public client: Client;

  constructor(client: Client) {
    this.client = client;
  }

  public async renderData(
    context: TurnContext,
    memory: Memory,
    tokenizer: Tokenizer,
    maxTokens: number
  ): Promise<RenderedPromptSection<string>> {
    const input = memory.getValue("temp.input") as string;
    const contentResults = [];
    const response = await this.client.api("/search/query").post({
      requests: [
        {
          entityTypes: ["driveItem"],
          query: {
            // Search for markdown files in the user's OneDrive and SharePoint
            // The supported file types are listed here:
            // https://learn.microsoft.com/sharepoint/technical-reference/default-crawled-file-name-extensions-and-parsed-file-types
            queryString: `${input} filetype:txt`,
          },
          // This parameter is required only when searching with application permissions
          // https://learn.microsoft.com/graph/search-concept-searchall
          // region: "US",
        },
      ],
    });
    for (const value of response?.value ?? []) {
      for (const hitsContainer of value?.hitsContainers ?? []) {
        contentResults.push(...(hitsContainer?.hits ?? []));
      }
    }

    // Add documents until you run out of tokens
    let length = 0,
      output = "";
    for (const result of contentResults) {
      const rawContent = await this.downloadSharepointFile(
        result.resource.webUrl
      );
      if (!rawContent) {
        continue;
      }
      let doc = `${rawContent}\n\n`;
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

  // Download the file from SharePoint
  // https://docs.microsoft.com/en-us/graph/api/driveitem-get-content
  private async downloadSharepointFile(
    contentUrl: string
  ): Promise<string | undefined> {
    const encodedUrl = this.encodeSharepointContentUrl(contentUrl);
    const fileContentResponse = await this.client
      .api(`/shares/${encodedUrl}/driveItem/content`)
      .responseType(ResponseType.TEXT)
      .get();

    return fileContentResponse;
  }

  private encodeSharepointContentUrl(webUrl: string): string {
    const byteData = Buffer.from(webUrl, "utf-8");
    const base64String = byteData.toString("base64");
    return (
      "u!" + base64String.replace("=", "").replace("/", "_").replace("+", "_")
    );
  }
}
```

## Build your own Data Ingestion

This doc showcases a solution to fully control the data ingestion process, including:

- **Load your source documents** - Besides text, if you have other types of documents, you may need to convert them to meaningful text, since the embedding model takes text as input.
- **Split into chunks** - The embedding model has input token limitation, so you may need to split documents into chunks to avoid API call failure.
- **Call embedding model** - Call the embedding model APIs to create embeddings for the given inputs.
- **Store embeddings** - Store the created embeddings into a vector database, also including useful metadata and raw content for further referencing.


### Sample Code

Here's a sample to create embeddings from source text document, and store into Azure AI Search Index:

- **loader.ts** - plain text as source input
  ```typescript
  import * as fs from "node:fs";

  export function loadTextFile(path: string): string {
    return fs.readFileSync(path, "utf-8");
  }
  ```

- **splitter.ts** - split text into chunks, with certain overlap
  ```typescript
  // split words by delimiters.
  const delimiters = [" ", "\t", "\r", "\n"];

  export function split(content: string, length: number, overlap: number): Array<string> {
    const results = new Array<string>();
    let cursor = 0, curChunk = 0;
    results.push("");
    while(cursor < content.length) {
      const curChar = content[cursor];
      if (delimiters.includes(curChar)) {
        // check chunk length
        while (curChunk < results.length && results[curChunk].length >= length) {
          curChunk ++;
        }
        for (let i = curChunk; i < results.length; i++) {
          results[i] += curChar;
        }
        if (results[results.length - 1].length >= length - overlap) {
          results.push("");
        }
      } else {
        // append
        for (let i = curChunk; i < results.length; i++) {
          results[i] += curChar;
        }
      }
      cursor ++;
    }
    while (curChunk < results.length - 1) {
      results.pop();
    }
    return results;
  }
  ```

- **embeddings.ts** - use Teams AI library `OpenAIEmbeddings` to create embeddings
  ```typescript
  import { OpenAIEmbeddings } from "@microsoft/teams-ai";

  const embeddingClient = new OpenAIEmbeddings({
    azureApiKey: "<your-aoai-key>",
    azureEndpoint: "<your-aoai-endpoint>",
    azureDeployment: "<your-embedding-deployment, e.g., text-embedding-ada-002>"
  });

  export async function createEmbeddings(content: string): Promise<number[]> {
    const response = await embeddingClient.createEmbeddings(content);
    return response.output[0];
  }
  ```

- **searchIndex.ts** - one-time and standalone method to create Azure AI Search Index
  ```typescript
  import { SearchIndexClient, AzureKeyCredential, SearchIndex } from "@azure/search-documents";

  const endpoint = "<your-search-endpoint>";
  const apiKey = "<your-search-key>";
  const indexName = "<your-index-name>";

  const indexDef: SearchIndex = {
    name: indexName,
    fields: [
      {
        type: "Edm.String",
        name: "id",
        key: true,
      },
      {
        type: "Edm.String",
        name: "content",
        searchable: true,
      },
      {
        type: "Edm.String",
        name: "filepath",
        searchable: true,
        filterable: true,
      },
      {
        type: "Collection(Edm.Single)",
        name: "contentVector",
        searchable: true,
        vectorSearchDimensions: 1536,
        vectorSearchProfileName: "default"
      }
    ],
    vectorSearch: {
      algorithms: [{
        name: "default",
        kind: "hnsw"
      }],
      profiles: [{
        name: "default",
        algorithmConfigurationName: "default"
      }]
    },
    semanticSearch: {
      defaultConfigurationName: "default",
      configurations: [{
        name: "default",
        prioritizedFields: {
          contentFields: [{
            name: "content"
          }]
        }
      }]
    }
  };

  export async function createNewIndex(): Promise<void> {
    const client = new SearchIndexClient(endpoint, new AzureKeyCredential(apiKey));
    await client.createIndex(indexDef);
  }
  ```

- **searchIndexer.ts** - upload created embeddings and other fields to Azure AI Search Index
  ```typescript
  import { AzureKeyCredential, SearchClient } from "@azure/search-documents";

  export interface Doc {
    id: string,
    content: string,
    filepath: string,
    contentVector: number[]
  }

  const endpoint = "<your-search-endpoint>";
  const apiKey = "<your-search-key>";
  const indexName = "<your-index-name>";
  const searchClient: SearchClient<Doc> = new SearchClient<Doc>(endpoint, indexName, new AzureKeyCredential(apiKey));

  export async function indexDoc(doc: Doc): Promise<boolean> {
    const response = await searchClient.mergeOrUploadDocuments([doc]);
    return response.results.every((result) => result.succeeded);
  }
  ```

- **index.ts** - orchestrate above components
  ```typescript
  import { createEmbeddings } from "./embeddings";
  import { loadTextFile } from "./loader";
  import { createNewIndex } from "./searchIndex";
  import { indexDoc } from "./searchIndexer";
  import { split } from "./splitter";

  async function main() {
    // Only need to call once
    await createNewIndex();

    // local files as source input
    const files = [`${__dirname}/data/A.md`, `${__dirname}/data/A.md`];
    for (const file of files) {
      // load file
      const fullContent = loadTextFile(file);

      // split into chunks
      const contents = split(fullContent, 1000, 100);
      let partIndex = 0;
      for (const content of contents) {
        partIndex ++;
        // create embeddings
        const embeddings = await createEmbeddings(content);

        // upload to index
        await indexDoc({
          id: `${file.replace(/[^a-z0-9]/ig, "")}___${partIndex}`,
          content: content,
          filepath: file,
          contentVector: embeddings,
        });
      }
    }
  }

  main().then().finally();
  ```
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>
