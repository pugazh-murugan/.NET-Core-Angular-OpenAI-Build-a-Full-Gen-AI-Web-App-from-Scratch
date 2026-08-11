# First RAG App — ASP.NET Core Web API Backend

Implements the architecture from the video: **Blob Storage → Azure AI Search (retriever) → Azure OpenAI GPT-4 (generator)**, exposed as a Web API for an Angular frontend.

## Project Structure
```
FirstRagApp/
├── Controllers/
│   ├── FileUploadController.cs   # POST api/fileupload -> stores file in Blob Storage
│   └── ChatController.cs         # POST api/chat -> retriever + LLM orchestration
├── Services/
│   ├── BlobStorageService.cs     # Azure.Storage.Blobs wrapper
│   ├── AzureSearchService.cs     # Azure.Search.Documents wrapper (the "retriever")
│   └── AzureOpenAIService.cs     # Azure.AI.OpenAI wrapper (the "generator")
├── Models/RagModels.cs
├── Program.cs                    # DI, CORS, Swagger
└── appsettings.json               # <-- fill in your Azure values here
```

## 1. Prerequisites (create in Azure Portal, as shown in the video)
1. **Storage Account** → create a container (e.g. `first-demo-app`).
2. **Azure AI Search** → "Import data" wizard, connect to the blob container, use the
   **default index name/fields** (`content`, `metadata_storage_name`, etc.), indexer schedule = 5 minutes.
3. **Azure OpenAI** → deploy a `gpt-4` model.

## 2. Configure `appsettings.json`
Replace every `REPLACE_...` placeholder:
- `AzureBlobStorage:ConnectionString` / `ContainerName` — from Storage Account → Access keys.
- `AzureSearch:Endpoint` / `IndexName` / `ApiKey` — from AI Search → Overview / Indexes / Keys.
- `AzureOpenAI:Endpoint` / `ApiKey` / `DeploymentName` — from Azure OpenAI → Keys and Endpoint.

**Better for production:** don't commit real keys to `appsettings.json`. Use `dotnet user-secrets`,
environment variables, or Azure Key Vault instead.

## 3. Run
```bash
dotnet restore
dotnet run
```
Swagger UI opens at `https://localhost:<port>/swagger` — test both endpoints directly there
before wiring up the Angular frontend.

## 4. Endpoints

### `POST /api/fileupload`  (multipart/form-data, field name: `file`)
Uploads a document to Blob Storage. The AI Search indexer picks it up automatically within ~5 minutes.

### `POST /api/chat`
```json
{
  "question": "Can you help me with GitHub Copilot features?",
  "documentNames": []   // optional: restrict to specific uploaded file names
}
```
Response:
```json
{
  "answer": "...",
  "sourceChunks": ["...retrieved passage 1...", "...retrieved passage 2..."],
  "model": "gpt-4"
}
```

## 5. Angular integration notes
- Set your Angular `environment.ts` `apiUrl` to this API's base URL (e.g. `https://localhost:5001/api`).
- Upload component: `POST` a `FormData` with the file to `/api/fileupload`.
- Chat component: `POST` `{ question, documentNames }` JSON to `/api/chat`.
- CORS is already configured in `Program.cs` for `http://localhost:4200` (default `ng serve` port).

## 6. Known simplification vs. the video
The video's default AI Search index (created via the portal wizard with a blob data source) uses
field names `content` and `metadata_storage_name`. If you customized your index schema, update the
two constants at the top of `AzureSearchService.cs` accordingly.
