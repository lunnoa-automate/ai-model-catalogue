# AI model catalogue

Public catalogue of LLM models that Lunnoa can seed. Each provider has its own folder. Lunnoa reads this repository when someone presses **Seed defaults**. It does not own the version history.

The default source is the `main` branch of this repository. A superadmin can point a deployment at another GitHub or GitLab repository that uses the same layout.

The list was refreshed on 13 November 2026. Models released before 13 November 2025 are not included. A model whose provider has already shut the endpoint down is not included either. Models that are still callable, but announced for shutdown, stay in the file with `isDeprecated` and `supportEndsAt`.

## Embedding models

Embedding models for knowledge notebooks are exempt from the release-date rule, because the ones that run well on-prem are older. The catalogue lists the multilingual models whose tokenizers ship in Lunnoa's Document Parser image:

- `bge-m3` (Ollama) and `BAAI/bge-m3` (Hugging Face model id, for self-hosted TEI or vLLM): 1024 dimensions, up to 8192 tokens.
- `intfloat/multilingual-e5-large` (Hugging Face model id): 1024 dimensions, up to 512 tokens.

Lunnoa only lists these models. The SuperAdmin still has to run a server that serves them and pick one as the default embedding model.

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
| `category` | `CHAT`, `REASONING`, `EMBEDDING`, `IMAGE_GEN`, `SPEECH`, `CODE`, or `MULTIMODAL`. |
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
