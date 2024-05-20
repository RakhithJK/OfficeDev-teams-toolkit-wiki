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
* [How to create a new RAG bot](#how-to-create-a-new-rag-bot)
* [How teams-ai helps to achieve RAG scenario](#how-teams-ai-helps-to-achieve-rag-scenario)
* [Choose between data sources](#choose-between-data-sources)
* [How to understand the RAG project]()

Customize the app template:
* [Customize Azure AI as Data Source](#azure-ai-search-as-data-source)
* [Customize Microsoft Graph & SharePoint as Data Source](#microsoft-365-as-data-source)
* [Build your own data ingestion](#build-your-own-data-ingestion)
* [Add more API for Custom API as data source](#add-more-api-for-custom-api-as-data-source)

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

<details open>
<summary> For Javascript language: </summary>

```typescript
// create OpenAIEmbeddings instance
const model = new OpenAIEmbeddings({ ... endpoint, apikey, model, ... });

// create embeddings for the given inputs
const embeddings = await model.createEmbeddings(model, inputs);

// your own logic to process embeddings
```

</details>


<details open>
<summary> For Python language: </summary>

```python
# create OpenAIEmbeddings instance
model = OpenAIEmbeddings(OpenAIEmbeddingsOptions(api_key, model))

# create embeddings for the given inputs
embeddings = await model.create_embeddings(inputs)

# your own logic to process embeddings
```

</details>




![Teams AI helps RAG](https://github.com/OfficeDev/TeamsFx/assets/13211513/7b1d14b1-ac05-4b2e-b8f6-7f2f8aab5e8f)
Teams-AI library also provides functionalities to ease each step of the retrieval and generation process.
- **Handle Input**: The most straightforward way is to pass user's input as is to retrieval. However, if you'd like to customize the input before retrieval, you can [add activity handler](./AddActivityHandlers.md) to certain incoming activities.

- **Retrieve data source**: Teams-AI library provides `DataSource` interface to let you to add your own retrieval logic. You will need to create your own `DataSource` instance, and the library orchestrator will call it on demand.
    

<details open>
<summary> For Javascript language: </summary>

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

</details>

<details open>
<summary> For Python language: </summary>

```python
class MyDataSource(DataSource):
  def __init__(self):
    self.name = "my_datasource_name"
  
  def name(self):
    return self.name

  async def render_data(self, _context: TurnContext, memory: Memory, tokenizer: Tokenizer, maxTokens: int):
    # your render data logic
```

</details>

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

- Register the data source in **`app.ts`**, 

  <details open>
  <summary> For Javascript language: </summary>

    ```typescript
    // Register your data source to prompt manager
    planner.prompts.addDataSource(new MyDataSource());
    ```

  </details>

  <details open>
  <summary> For Python language: </summary>

    ```python
    planner.prompts.add_data_source(MyDataSource())
    ```

  </details>

- Create **`prompts/qa/skprompt.txt`** for prompt template text.
  ```text
  The following is a conversation with an AI assistant. The assistant is helpful, creative, clever, and very friendly to answer user's question.

  Base your answer off the text below:
  ```

- Create **`prompts/qa/config.json`** to connect with the data source.
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

- `Customize` allows you to fully control the data ingestion, see the sample on [Build your own Data Ingestion](#build-your-own-data-ingestion) to build your own vector index, and use it as data source. There are other alternatives, e.g., Azure Cosmos DB Vector Database Extension or Azure PostgreSQL Server pgvector Extension as vector databases, or Bing Web Search API to get latest web content. You may implement any `DataSource` instance to connect with your own data source.
- [`Azure AI Search`](#azure-ai-search-as-data-source) provides a sample to add your documents to Azure AI Search Service, then use the search index as data source.

- `Custom API` allows your chatbot can invoke the API defined in the OpenAPI description document to retrieve domain data from API service.

- [Microsoft Graph & SharePoint](#microsoft-graph-search-api-as-data-source) provides a sample to use M365 content from Microsoft Graph Search API as data source.

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Build your own Data Ingestion

This doc showcases a solution to fully control the data ingestion process, including:

- **Load your source documents** - Besides text, if you have other types of documents, you may need to convert them to meaningful text, since the embedding model takes text as input.
- **Split into chunks** - The embedding model has input token limitation, so you may need to split documents into chunks to avoid API call failure.
- **Call embedding model** - Call the embedding model APIs to create embeddings for the given inputs.
- **Store embeddings** - Store the created embeddings into a vector database, also including useful metadata and raw content for further referencing.


### Sample Code

Here's a sample to create embeddings from source text document, and store into Azure AI Search Index:

<details>
<summary> For Javascript language: </summary>

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

</details>

<details>
<summary> For Python language: </summary>

- **loader.py** - plain text as source input
```python
def load_text_file(path: str) -> str:
    with open(path, 'r', encoding='utf-8') as file:
        return file.read()
```

- **splitter.py** - split text into chunks, with certain overlap
```python
def split(content: str, length: int, overlap: int) -> list[str]:
    delimiters = [" ", "\t", "\r", "\n"]
    results = [""]
    cursor = 0
    cur_chunk = 0
    while cursor < len(content):
        cur_char = content[cursor]
        if cur_char in delimiters:
            while cur_chunk < len(results) and len(results[cur_chunk]) >= length:
                cur_chunk += 1
            for i in range(cur_chunk, len(results)):
                results[i] += cur_char
            if len(results[-1]) >= length - overlap:
                results.append("")
        else:
            for i in range(cur_chunk, len(results)):
                results[i] += cur_char
        cursor += 1
    while cur_chunk < len(results) - 1:
        results.pop()
    return results
```

- **embeddings.py** - use Teams AI library OpenAIEmbeddings to create embeddings
```python
async def create_embeddings(text: str, embeddings):
    result = await embeddings.create_embeddings(text)
    
    return result.output[0]
```

- **search_index.py** - one-time and standalone method to create Azure AI Search Index
```python
async def create_index_if_not_exists(client: SearchIndexClient, name: str):
    doc_index = SearchIndex(
        name=name,
        fields = [
            SimpleField(name="docId", type=SearchFieldDataType.String, key=True),
            SimpleField(name="docTitle", type=SearchFieldDataType.String),
            SearchableField(name="description", type=SearchFieldDataType.String, searchable=True),
            SearchField(name="descriptionVector", type=SearchFieldDataType.Collection(SearchFieldDataType.Single), searchable=True, vector_search_dimensions=1536, vector_search_profile_name='my-vector-config'),
        ],
        scoring_profiles=[],
        cors_options=CorsOptions(allowed_origins=["*"]),
        vector_search = VectorSearch(
            profiles=[VectorSearchProfile(name="my-vector-config", algorithm_configuration_name="my-algorithms-config")],
            algorithms=[HnswAlgorithmConfiguration(name="my-algorithms-config")],
        )
    )

    client.create_or_update_index(doc_index)
```

- **search_indexer.py** - upload created embeddings and other fields to Azure AI Search Index
```python
from embeddings import create_embeddings
from search_index import create_index_if_not_exists
from loader import load_text_file
from split import split

async def get_doc_data(embeddings):
    file_path=f'{os.getcwd()}/my_file_path_1'
    raw_description1 = split(load_text_file(file_path), 1000, 100)
    doc1 = {
        "docId": "1",
        "docTitle": "my_titile_1",
        "description": raw_description1,
        "descriptionVector": await create_embeddings(raw_description1, embeddings=embeddings),
    }
    
    file_path=f'{os.getcwd()}/my_file_path_2'
    raw_description2 = split(load_text_file(file_path), 1000, 100)
    doc2 = {
        "docId": "2",
        "docTitle": "my_titile_2",
        "description": raw_description2,
        "descriptionVector": await create_embeddings(raw_description2, embeddings=embeddings),
    }

    return [doc1, doc2]

async def setup(search_api_key, search_api_endpoint):
    index = 'my_index_name'
    credentials = AzureKeyCredential(search_api_key)
    search_index_client = SearchIndexClient(search_api_endpoint, credentials)
    await create_index_if_not_exists(search_index_client, index)

    search_client = SearchClient(search_api_endpoint, index, credentials)
    embeddings=AzureOpenAIEmbeddings(AzureOpenAIEmbeddingsOptions(
          azure_api_key="<your-aoai-key>",
          azure_endpoint="<your-aoai-endpoint>",
          azure_deployment="<your-embedding-deployment, e.g., text-embedding-ada-002>"
    ))
    data = await get_doc_data(embeddings=embeddings)
    await search_client.merge_or_upload_documents(data)
```

- **index.py** - orchestrate above components
```python
from search_indexer import setup

search_api_key = '<your-key>'
search_api_endpoint = '<your-endpoint>'
asyncio.run(setup(search_api_key, search_api_endpoint))
```

</details>
<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

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

<details>
<summary> For Javascript language: </summary>

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

</details>

<details>
<summary> For Python language: </summary>

```python
async def get_embedding_vector(text: str):
    embeddings = AzureOpenAIEmbeddings(AzureOpenAIEmbeddingsOptions(
        azure_api_key='<your-aoai-key>',
        azure_endpoint='<your-aoai-endpoint>',
        azure_deployment='<your-aoai-embedding-deployment>'
    ))
    
    result = await embeddings.create_embeddings(text)
    if (result.status != 'success' or not result.output):
        raise Exception(f"Failed to generate embeddings for description: {text}")
    
    return result.output[0]

@dataclass
class Doc:
    docId: Optional[str] = None
    docTitle: Optional[str] = None
    description: Optional[str] = None
    descriptionVector: Optional[List[float]] = None

@dataclass
class MyDataSourceOptions:
    name: str
    indexName: str
    azureAISearchApiKey: str
    azureAISearchEndpoint: str

from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient
import json

@dataclass
class Result:
    def __init__(self, output, length, too_long):
        self.output = output
        self.length = length
        self.too_long = too_long

class MyDataSource(DataSource):
    def __init__(self, options: MyDataSourceOptions):
        self.name = options.name
        self.options = options
        self.searchClient = SearchClient(
            options.azureAISearchEndpoint,
            options.indexName,
            AzureKeyCredential(options.azureAISearchApiKey)
        )
        
    def name(self):
        return self.name

    async def render_data(self, _context: TurnContext, memory: Memory, tokenizer: Tokenizer, maxTokens: int):
        query = memory.get('temp.input')
        embedding = await get_embedding_vector(query)
        vector_query = VectorizedQuery(vector=embedding, k_nearest_neighbors=2, fields="descriptionVector")

        if not query:
            return Result('', 0, False)

        selectedFields = [
            'docTitle',
            'description',
            'descriptionVector',
        ]

        searchResults = self.searchClient.search(
            search_text=query,
            select=selectedFields,
            vector_queries=[vector_query],
        )

        if not searchResults:
            return Result('', 0, False)

        usedTokens = 0
        doc = ''
        for result in searchResults:
            tokens = len(tokenizer.encode(json.dumps(result["description"])))

            if usedTokens + tokens > maxTokens:
                break

            doc += json.dumps(result["description"])
            usedTokens += tokens

        return Result(doc, usedTokens, usedTokens > maxTokens)
```

</details>

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

### Integrate vectorization

This section will show you how to create vectorized data on Azure AI Search and run a series of queries in Python language.

> Note: For more details, you can refer to this [demo sample](https://github.com/Azure/azure-search-vector-samples/blob/main/demo-python/code/integrated-vectorization/azure-search-integrated-vectorization-sample.ipynb).

* **Connect to Blob Storage and load documents**

  Retrieve documents from Blob Storage. Upload your local documents.

  You need to prepare a **blob connection** in your Azure AI Search service, and get your `blob_connection_string` and `blob_container_name`.
    ```python
    from azure.storage.blob import BlobServiceClient  
    import os

    # Connect to Blob Storage
    blob_service_client = BlobServiceClient.from_connection_string(blob_connection_string)
    container_client = blob_service_client.get_container_client(blob_container_name)
    if not container_client.exists():
        container_client.create_container()

    documents_directory = os.path.join("<your-local-doc-path>")
    for file in os.listdir(documents_directory):
        with open(os.path.join(documents_directory, file), "rb") as data:
            name = os.path.basename(file)
            if not container_client.get_blob_client(name).exists():
                container_client.upload_blob(name=name, data=data)
    ```
* **Create a blob data source connector on Azure AI Search**
    ```python
    from azure.search.documents.indexes import SearchIndexerClient
    from azure.search.documents.indexes.models import (
        SearchIndexerDataContainer,
        SearchIndexerDataSourceConnection
    )
    from azure.search.documents.indexes._generated.models import NativeBlobSoftDeleteDeletionDetectionPolicy

    # Create a data source 
    indexer_client = SearchIndexerClient(endpoint, credential)
    container = SearchIndexerDataContainer(name=blob_container_name)
    data_source_connection = SearchIndexerDataSourceConnection(
        name=f"{index_name}-blob",
        type="azureblob",
        connection_string=blob_connection_string,
        container=container,
        data_deletion_detection_policy=NativeBlobSoftDeleteDeletionDetectionPolicy()
    )
    data_source = indexer_client.create_or_update_data_source_connection(data_source_connection)
    ```
* **Create a search index**

  This step is similar to the step in [Build your own Data Ingestion/Sample Code(Python language)/search_index.py](#build-your-own-data-ingestion). But vector and nonvector content is stored in a search index, and we will create a different search configuration.
    ```python
    # Create a search index  
    index_client = SearchIndexClient(endpoint=endpoint, credential=credential)  
    fields = [  
        SearchField(name="parent_id", type=SearchFieldDataType.String, sortable=True, filterable=True, facetable=True),  
        SearchField(name="title", type=SearchFieldDataType.String),  
        SearchField(name="chunk_id", type=SearchFieldDataType.String, key=True, sortable=True, filterable=True, facetable=True, analyzer_name="keyword"),  
        SearchField(name="chunk", type=SearchFieldDataType.String, sortable=False, filterable=False, facetable=False),  
        SearchField(name="vector", type=SearchFieldDataType.Collection(SearchFieldDataType.Single), vector_search_dimensions=1536, vector_search_profile_name="myHnswProfile"),  
    ]  
    
    # Configure the vector search configuration  
    vector_search = VectorSearch(  
        algorithms=[  
            HnswAlgorithmConfiguration(  
                name="myHnsw",  
                parameters=HnswParameters(  
                    m=4,  
                    ef_construction=400,  
                    ef_search=500,  
                    metric=VectorSearchAlgorithmMetric.COSINE,  
                ),  
            ),  
            ExhaustiveKnnAlgorithmConfiguration(  
                name="myExhaustiveKnn",  
                parameters=ExhaustiveKnnParameters(  
                    metric=VectorSearchAlgorithmMetric.COSINE,  
                ),  
            ),  
        ],  
        profiles=[  
            VectorSearchProfile(  
                name="myHnswProfile",  
                algorithm_configuration_name="myHnsw",  
                vectorizer="myOpenAI",  
            ),  
            VectorSearchProfile(  
                name="myExhaustiveKnnProfile",  
                algorithm_configuration_name="myExhaustiveKnn",  
                vectorizer="myOpenAI",  
            ),  
        ],  
        vectorizers=[  
            AzureOpenAIVectorizer(  
                name="myOpenAI",  
                kind="azureOpenAI",  
                azure_open_ai_parameters=AzureOpenAIParameters(  
                    resource_uri=azure_openai_endpoint,  
                    deployment_id=azure_openai_embedding_deployment,  
                    api_key=azure_openai_key,  
                ),  
            ),  
        ],  
    )  
    
    semantic_config = SemanticConfiguration(  
        name="my-semantic-config",  
        prioritized_fields=SemanticPrioritizedFields(  
            content_fields=[SemanticField(field_name="chunk")]  
        ),  
    )  
    
    # Create the semantic search with the configuration  
    semantic_search = SemanticSearch(configurations=[semantic_config])  
    
    # Create the search index
    index = SearchIndex(name=index_name, fields=fields, vector_search=vector_search, semantic_search=semantic_search)  
    result = index_client.create_or_update_index(index)
    ```
* **Create a skillset**

  Skills drive integrated vectorization. [Text Split](https://learn.microsoft.com/azure/search/cognitive-search-skill-textsplit) provides data chunking. [AzureOpenAIEmbedding](https://learn.microsoft.com/azure/search/cognitive-search-skill-azure-openai-embedding) handles calls to Azure OpenAI, using the connection information you provide in the environment variables. An [indexer projection](https://learn.microsoft.com/azure/search/index-projections-concept-intro) specifies secondary indexes used for chunked data.
    ```python
    # Create a skillset  
    skillset_name = f"{index_name}-skillset"  
    
    split_skill = SplitSkill(  
        description="Split skill to chunk documents",  
        text_split_mode="pages",  
        context="/document",  
        maximum_page_length=2000,  
        page_overlap_length=500,  
        inputs=[  
            InputFieldMappingEntry(name="text", source="/document/content"),  
        ],  
        outputs=[  
            OutputFieldMappingEntry(name="textItems", target_name="pages")  
        ],  
    )  
    
    embedding_skill = AzureOpenAIEmbeddingSkill(  
        description="Skill to generate embeddings via Azure OpenAI",  
        context="/document/pages/*",  
        resource_uri=azure_openai_endpoint,  
        deployment_id=azure_openai_embedding_deployment,  
        api_key=azure_openai_key,  
        inputs=[  
            InputFieldMappingEntry(name="text", source="/document/pages/*"),  
        ],  
        outputs=[  
            OutputFieldMappingEntry(name="embedding", target_name="vector")  
        ],  
    )  
    
    index_projections = SearchIndexerIndexProjections(  
        selectors=[  
            SearchIndexerIndexProjectionSelector(  
                target_index_name=index_name,  
                parent_key_field_name="parent_id",  
                source_context="/document/pages/*",  
                mappings=[  
                    InputFieldMappingEntry(name="chunk", source="/document/pages/*"),  
                    InputFieldMappingEntry(name="vector", source="/document/pages/*/vector"),  
                    InputFieldMappingEntry(name="title", source="/document/metadata_storage_name"),  
                ],  
            ),  
        ],  
        parameters=SearchIndexerIndexProjectionsParameters(  
            projection_mode=IndexProjectionMode.SKIP_INDEXING_PARENT_DOCUMENTS  
        ),  
    )  
    
    skillset = SearchIndexerSkillset(  
        name=skillset_name,  
        description="Skillset to chunk documents and generating embeddings",  
        skills=[split_skill, embedding_skill],  
        index_projections=index_projections,  
    )  
    
    client = SearchIndexerClient(endpoint, credential)  
    client.create_or_update_skillset(skillset)
    ```
* **Create an indexer**
    ```python
    # Create an indexer  
    indexer_name = f"{index_name}-indexer"  
    
    indexer = SearchIndexer(  
        name=indexer_name,  
        description="Indexer to index documents and generate embeddings",  
        skillset_name=skillset_name,  
        target_index_name=index_name,  
        data_source_name=data_source.name,  
        # Map the metadata_storage_name field to the title field in the index to display the PDF title in the search results  
        field_mappings=[FieldMapping(source_field_name="metadata_storage_name", target_field_name="title")]  
    )  
    
    indexer_client = SearchIndexerClient(endpoint, credential)  
    indexer_result = indexer_client.create_or_update_indexer(indexer)  
    
    # Run the indexer  
    indexer_client.run_indexer(indexer_name)
    ```
* **Perform a vector similarity search**

  This example shows a pure vector search using the vectorizable text query, all you need to do is pass in text and your vectorizer will handle the query vectorization.
  If you indexed the health plan PDF file, send queries that ask plan-related questions.
    ```python
    # Pure Vector Search
    query = "<your-query>"  
    
    search_client = SearchClient(endpoint, index_name, credential=credential)
    vector_query = VectorizableTextQuery(text=query, k_nearest_neighbors=1, fields="vector", exhaustive=True)
    # Use the below query to pass in the raw vector query instead of the query vectorization
    # vector_query = RawVectorQuery(vector=generate_embeddings(query), k_nearest_neighbors=3, fields="vector")
    
    results = search_client.search(  
        search_text=None,  
        vector_queries= [vector_query],
        select=["parent_id", "chunk_id", "chunk"],
        top=1
    )  
    
    for result in results:  
        print(f"parent_id: {result['parent_id']}")  
        print(f"chunk_id: {result['chunk_id']}")  
        print(f"Score: {result['@search.score']}")  
        print(f"Content: {result['chunk']}")
    ```

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Add more API for Custom API as data source
You can follow the following steps to extend the Custom Copilot from Custom API template with more APIs.

1. Update `./appPackage/apiSpecificationFile/openapi.*`

    Copy corresponding part of the API you want to add from your spec, and append to `./appPackage/apiSpecificationFile/openapi.*`.

1. Update `./src/prompts/chat/actions.json`

    Fill necessary info and properties for path, query and/or body for the API in the following object, and add it in the array in `./src/prompts/chat/actions.json`.
    ```
    {
      "name": "${{YOUR-API-NAME}}",
      "description": "${{YOUR-API-DESCRIPTION}}",
      "parameters": {
        "type": "object",
        "properties": {
          "query": {
            "type": "object",
            "properties": {
              "${{YOUR-PROPERTY-NAME}}": {
                "type": "${{YOUR-PROPERTY-TYPE}}",
                "description": "${{YOUR-PROPERTY-DESCRIPTION}}",
              }
              // You can add more query properties here
            }
          },
          "path": {
            // Same as query properties
          },
          "body": {
            // Same as query properties
          }
        }
      }
    }
    ```

1. Update `./src/adaptiveCards`

    Create a new file with name `${{YOUR-API-NAME}}.json`, and fill in the adaptive card for the API response of your API.

1. Update `./src/app/app.js`

    Add following code before `module.exports = app;`. Remember to replace necessary info.

    ```
    app.ai.action(${{YOUR-API-NAME}}, async (context: TurnContext, state: ApplicationTurnState, parameter: any) => {
      const client = await api.getClient();
      
      const path = client.paths[${{YOUR-API-PATH}}];
      if (path && path.${{YOUR-API-METHOD}}) {
        const result = await path.${{YOUR-API-METHOD}}(parameter.path, parameter.body, {
          params: parameter.query,
        });
        const card = generateAdaptiveCard("../adaptiveCards/${{YOUR-API-NAME}}.json", result);
        await context.sendActivity({ attachments: [card] });
      } else {
        await context.sendActivity("no result");
      }
      return "result";
    });
    ```

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>

## Microsoft 365 as Data Source

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

<p align="right"><a href="#in-this-tutorial-you-will-learn">back to top</a></p>
