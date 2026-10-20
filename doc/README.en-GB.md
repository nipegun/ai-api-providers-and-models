# AI API Providers and Models

This README is also available in:\
en-GB, [en-US](../README.md), [es-AR](README.es-AR.md), [es-ES](README.es-ES.md)

A shared catalogue of AI API providers and models for applications that need a common source of connection settings, model capabilities, token limits, and reference prices.

The project maintains two JSON files. Applications can use them to populate provider and model selectors or configure their API integrations, keeping catalogue updates centralised.

## Contents

| File | Purpose |
| --- | --- |
| [api-providers.json](../api-providers.json) | Provider names, API URLs, request formats, and integration options. |
| [api-models.json](../api-models.json) | Models grouped by provider, with identifiers, capabilities, limits, reasoning settings, prices, and source links. |
| [MANUAL.en-GB.md](MANUAL.en-GB.md) | Instructions for browsing, using, and updating the catalogue. |
| [CODE.en-GB.md](CODE.en-GB.md) | Data structure and integration reference. |

The catalogue includes direct providers, gateways, and local services. Examples include Anthropic, Gemini, OpenAI, DeepSeek, OpenRouter, Vercel AI Gateway, Ollama, llama.cpp, and vLLM. Some providers have an empty model list.

The shared repository location specified for this project is [nipegun/ai-api-providers-and-models](https://github.com/nipegun/ai-api-providers-and-models).

## Getting started on Debian

1. Save `api-providers.json` and `api-models.json` in the same directory, keeping their names.
2. Configure your consuming application to read both files from that directory.
3. To explore the files in a terminal, optionally install `jq`. Run these package installation commands as `root`:

```sh
apt-get update
apt-get install -y jq
```

The catalogue itself has no package dependencies, build step, database, or service to start. Any application with a JSON reader can use it. The commands below assume your terminal is in the directory containing both files and can be run as a regular user.

## Usage

List provider identifiers:

```sh
jq -r 'keys[]' api-providers.json
```

Inspect a provider's connection settings:

```sh
jq '.openai' api-providers.json
```

List the model identifiers recorded for that provider:

```sh
jq -r '.openai[].name' api-models.json
```

Replace `openai` with another provider key as needed. These commands read local data; they do not make model requests.

To use a model in an application, join the two files by provider key, select its exact `name`, and use the provider's `format` together with the model's `api_type` to choose the appropriate integration. Credentials, HTTP requests, response handling, and local model installation belong to the consuming application. This project does not include an API client or inference server.

## Understanding the data

- **Provider keys connect the files.** For example, `api-providers.json["openai"]` describes the provider whose models appear in `api-models.json["openai"]`. A model identifier belongs to its provider; entries from different providers are separate records.
- **Most model values are strings.** This includes token counts, prices, and flags such as `"true"` and `"false"`. Preserve their types in the files. The optional `reasoning_effort_map` is an object with numeric values.
- **An empty string means no value is recorded.** It does not mean zero, free access, unlimited capacity, or lack of support. An empty model array means no models are listed for that provider.
- **Reasoning settings need interpretation.** Effort levels are stored as a comma-separated string. Empty reasoning settings can coexist with `reasoning: "true"`.
- **Prices are reference values in USD per million tokens.** The schema stores one input, cached-input, and output price per model entry. It does not represent every pricing tier, route, promotion period, access condition, or additional charge. A recorded `"0"` does not by itself imply unrestricted free access.
- **Connection details are metadata.** Local URLs require a separately running service. URL placeholders such as `{account_id}` must be resolved by the consuming application.

See the [technical reference](CODE.en-GB.md) for the complete field descriptions and the [manual](MANUAL.en-GB.md) for practical examples.

## Maintenance

Updates are maintained centrally and shared with consuming applications. Preserve the filenames, provider keys, field names, JSON structure, and value types when editing the catalogue.

Use the source links recorded in model entries to confirm changes. When a provider announces a model's deprecation or retirement, remove that entry immediately, even if the effective date is later. Apply the announcement to the affected provider; it does not automatically establish retirement through other providers.

Consumers should refresh their copies deliberately and review model selections after an update. The repository contains no automatic updater, synchronization service, or scheduled availability checks.
