# AI model catalogue

Public catalogue of LLM models that Lunnoa can seed. Each provider has its own folder. Lunnoa reads this repository when someone presses **Seed defaults**. It does not own the version history.

The default source is the `main` branch of this repository. A superadmin can point a deployment at another GitHub or GitLab repository that uses the same layout.

The list was refreshed on 13 November 2026. Models released before 13 November 2025 are not included. A model whose provider has already shut the endpoint down is not included either. Models that are still callable, but announced for shutdown, stay in the file with `isDeprecated` and `supportEndsAt`.

## Embedding models

Embedding models for knowledge notebooks are exempt from the release-date rule, because the ones that run well on-prem are older. The catalogue lists the multilingual models whose tokenizers ship in Lunnoa's Document Parser image:

- `bge-m3` (Ollama) and `BAAI/bge-m3` (Hugging Face model id, for self-hosted TEI or vLLM): 1024 dimensions, up to 8192 tokens.
- `intfloat/multilingual-e5-large` (Hugging Face model id): 1024 dimensions, up to 512 tokens.

Lunnoa only lists these models. The SuperAdmin still has to run a server that serves them and pick one as the default embedding model.

### The `embedding` block

`EMBEDDING` models can carry an optional `embedding` object. Lunnoa uses it to offer only vector sizes the model supports, to reject other sizes, and to pick the tokenizer that cuts document chunks.

```json
"embedding": {
  "defaultDimensions": 1024,
  "minDimensions": 32,
  "maxDimensions": 1024,
  "tokenizer": "Qwen/Qwen3-Embedding-0.6B"
}
```

| Field | Meaning |
| --- | --- |
| `defaultDimensions` | Size preselected in Lunnoa. Required. |
| `minDimensions`, `maxDimensions` | The model accepts any size in this range. Lunnoa sends the chosen size as `dimensions` on embed calls. |
| `allowedDimensions` | The model only produces these sizes. With one size, Lunnoa sends no `dimensions` value. |
| `tokenizer` | Hugging Face repo id of the tokenizer for chunking. Lunnoa uses it only when the Document Parser image has it baked in. Models that share a tokenizer name the same repo. |

Give either `minDimensions` and `maxDimensions` or `allowedDimensions`, not both. `defaultDimensions` must be inside the range or in the list.

Lunnoa versions without this field ignore it. A malformed block is ignored with a warning and the model still syncs.

Cloud embedding models, which send document text to the provider:

- `Qwen/Qwen3-Embedding-0.6B`, `Qwen/Qwen3-Embedding-4B` and `Qwen/Qwen3-Embedding-8B` (Featherless, `/v1/embeddings`): multilingual, up to 1024, 2560 and 4096 dimensions, up to 32768 tokens. The three models share one tokenizer, `Qwen/Qwen3-Embedding-0.6B`, which ships in the Document Parser image since lunnoa_automate#641. Use them only where sending documents to Featherless is allowed. The same open weights can be self-hosted instead.

## Re-rank models

Re-rank models (category `RERANK`) re-order knowledge search results before they reach the chat model. They are exempt from the release-date rule for the same reason as embedding models. The catalogue lists:

- `BAAI/bge-reranker-v2-m3` (Hugging Face model id, for self-hosted TEI, vLLM or Infinity): multilingual cross-encoder, up to 8192 tokens.

### The `rerank` block

`RERANK` models can carry an optional `rerank` object. Lunnoa uses it to cut passages the model cannot read, instead of the re-rank server cutting them silently, and to split large candidate lists into several calls.

```json
"rerank": {
  "maxInputTokens": 8192,
  "multilingual": true,
  "maxDocumentsPerCall": 32
}
```

| Field | Meaning |
| --- | --- |
| `maxInputTokens` | Query plus one passage, in model tokens. Required. Lunnoa cuts each passage to fit before the call. |
| `multilingual` | The model ranks languages other than English well. Shown on the SuperAdmin re-rank card. |
| `maxDocumentsPerCall` | Passages per API call. Lunnoa sends more in several calls and merges the scores. `32` matches the default `--max-client-batch-size` of TEI. |

Lunnoa versions without this field ignore it. A malformed block is ignored with a warning and the model still syncs.

Only catalogue models can be selected as the platform re-ranker. The SuperAdmin runs the server and picks the model in Admin → System Settings → Knowledge.

Lunnoa versions without the `RERANK` category skip these models with a warning when they read the catalogue. Versions older than that reject the whole catalogue, so deploy a version that knows `RERANK` before you merge a `RERANK` model.

## Layout

`catalogue.json` lists the providers. Each provider folder contains `models.json`.

```
catalogue.json
openai/models.json
anthropic/models.json
gemini/models.json
```

A model is identified by `providerSlug` + `modelId`. That pair is the match key. Do not rename `modelId` to mean "this is the successor." Add the new model and point the old one at it with `replacedByModelId`.

## Deprecation

Set these on a model that should stay in the catalogue while people move off it:

- `isDeprecated` — show the model as deprecated. A `supportEndsAt` date implies this even when the flag is false.
- `supportEndsAt` — calendar date (`YYYY-MM-DD`) when catalogue support ends. The model keeps working after that date. Lunnoa shows the date and does not cut the model off.
- `replacedByModelId` — `modelId` of the replacement in the same provider file.
- `deprecationMessage` — optional sentence shown instead of the generated one.

Example:

```json
{
  "modelId": "gemini-3-pro-preview",
  "isDeprecated": true,
  "supportEndsAt": "2026-03-09",
  "replacedByModelId": "gemini-3.1-pro-preview",
  "deprecationMessage": "Shut down March 9, 2026 per Google. Use gemini-3.1-pro-preview instead."
}
```

Removing a model from its file is how it leaves the catalogue. Lunnoa keeps the row when a connection or agent still uses it, and marks that row deprecated. Admin-created models are not in this repository and are left alone.

## Fields

Boolean flags are stored explicitly. Token limits are omitted when unknown.

| Field | Meaning |
| --- | --- |
| `modelId` | Provider id sent to the API. |
| `name` | Label shown in Lunnoa. |
| `description` | Short explanation. |
| `category` | `CHAT`, `REASONING`, `EMBEDDING`, `RERANK`, `IMAGE_GEN`, `SPEECH`, `CODE`, or `MULTIMODAL`. |
| `supportsVision` | Accepts image input. |
| `supportsTools` | Can call tools. |
| `supportsTextStreaming` | Streams text. |
| `supportsToolStreaming` | Streams tool calls. |
| `supportsTemperature` | Accepts a temperature value. |
| `supportsReasoning` | Has a reasoning mode. |
| `temperatureTopPExclusive` | Temperature and top-p cannot be sent together. |
| `modalities` | Input types, such as `text` and `image`. |
| `maxContextTokens` | Context window. |
| `maxOutputTokens` | Maximum output length. |
| `sortOrder` | Order within the provider. Lower comes first. |
| `isRecommended` | Offered as a default choice. |
