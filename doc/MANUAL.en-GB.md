# Catalogue user manual

This MANUAL is also available in:\
en-GB, [en-US](MANUAL.md), [es-AR](MANUAL.es-AR.md), [es-ES](MANUAL.es-ES.md)

## Start here

Save `api-providers.json` and `api-models.json` together. They can be opened in a text editor or read by an application. For terminal examples, install `jq` using the [Debian instructions](README.en-GB.md#getting-started-on-debian) and open a terminal in the directory containing both files.

The catalogue does not require an API key to read. Calling a hosted model requires the credentials and access conditions of the selected service. Local services must be installed and started separately.

## Find a provider and its models

Show provider keys alongside their display names:

```sh
jq -r 'to_entries[] | [.key, .value.name] | @tsv' api-providers.json
```

Use a key from the first column to inspect its configuration and list its models:

```sh
jq '.openai' api-providers.json
jq -r '.openai[].name' api-models.json
```

To display all recorded model details for that provider:

```sh
jq '.openai[]' api-models.json
```

Keep the exact provider key and model `name`, including capitalisation, punctuation, slashes, and suffixes. The same model offered by another provider may have a different identifier, price, limit, or configuration.

## Explore capabilities and limits

Show provider, model, context window, and maximum output as tab-separated columns:

```sh
jq -r 'to_entries[] | .key as $vProvider | .value[] | [$vProvider, .name, .tokens_max_context_window, .tokens_max_output] | @tsv' api-models.json
```

List model entries marked as supporting tools:

```sh
jq -r 'to_entries[] | .key as $vProvider | .value[] | select(.tool_calling == "true") | [$vProvider, .name] | @tsv' api-models.json
```

Compare flags to the string `"true"`, not to the JSON boolean `true`. A blank limit is unspecified, not unlimited. Context and output limits are distinct; do not assume the full context window and full output allowance can be used simultaneously.

For reasoning configuration, inspect `reasoning_mode`, `reasoning_effort_levels`, `reasoning_effort_default`, and the optional `reasoning_effort_map`. Empty settings do not override a recorded `reasoning: "true"`. The consuming application must translate these values into the selected provider's request format.

## Read prices

Display the model identifier followed by input, cached-input, and output prices for a provider:

```sh
jq -r '.openrouter[] | [.name, .price_per_million_tokens_input, .price_per_million_tokens_cached_input, .price_per_million_tokens_output] | @tsv' api-models.json
```

Recorded prices use USD per million tokens. A blank field is unknown or unrecorded; `"0"` is an explicit zero. Cached-input prices do not represent every cache-related charge.

The catalogue does not encode all context tiers, route differences, promotion end dates, request modes, or access conditions. Consult the entry's `model_info_url` and `provider_models_url` before relying on a price or limit for a particular request.

## Connect an application

1. Load both JSON files and select a provider key present in each.
2. Choose a model from that provider's array. An empty array supplies no model choices.
3. Read the provider's `format` and the model's `api_type`. Where applicable, `url_responses` supplies the Responses endpoint; `url` contains the provider's primary URL. Some integrations need to construct a complete URL from it.
4. Supply credentials through the application's own configuration. Resolve placeholders such as Cloudflare's `{account_id}`. The `credential_format` value describes the expected credential layout; it is not a credential.
5. Apply supported limits, reasoning, temperature, and tool settings through the application's provider adapter.
6. Let the application send the request and process the response.

See the [technical reference](CODE.en-GB.md) for field meanings. The repository supplies the data for this workflow; it does not implement the workflow itself.

## Keep a copy current

Obtain both catalogue files from the same update and replace the consuming application's copies together. Reload or restart that application if it caches the files; the mechanism depends on the application.

If a previously selected model disappears, choose another listed model for that provider and update the application's selection. Announced deprecations and retirements are removed immediately, even when a service temporarily redirects requests for the old identifier.

Maintain shared data in this project, preserve the existing schema, and distribute updates to consumers. Keep deployment-specific credentials and local URL overrides in the consuming application's configuration.

## Troubleshooting

| Symptom | Explanation or next step |
| --- | --- |
| `jq: command not found` | Install the optional `jq` package following the README. |
| File not found | Open the terminal in the directory containing the JSON files or use their full paths. |
| No models are printed | Inspect the provider key and its array. An existing provider can have an empty list. |
| A value or price is blank | No value is recorded. Consult the source links instead of treating it as zero. |
| A local URL does not respond | Start the corresponding local service and match its address in the consuming application's configuration. |
| A model request fails | Review the exact model identifier, provider access, credentials, endpoint, and request format in the consuming application. |
