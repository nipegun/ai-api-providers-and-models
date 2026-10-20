# Catalogue technical reference

This CODE is also available in:\
en-GB, [en-US](CODE.md), [es-AR](CODE.es-AR.md), [es-ES](CODE.es-ES.md)

## Index

- [Architecture and design](#architecture-and-design)
- [Module map](#module-map)
- [Key data symbols](#key-data-symbols)
- [Provider fields](#provider-fields)
- [Model fields](#model-fields)
- [Main flows](#main-flows)
- [Entry points](#entry-points)
- [Impact analysis](#impact-analysis)
- [Extension points](#extension-points)

## Architecture and design

This is a data repository. It has no executable application code, HTTP handlers, classes, or public functions. Its interface consists of two JSON documents:

- `api-providers.json`: an object mapping provider keys to configuration objects.
- `api-models.json`: an object mapping the same provider keys to arrays of model objects.

Separating connection settings from models avoids repeating provider configuration in every model entry. Keeping both files as a shared catalogue lets independent applications consume the same updates without maintaining separate authoritative datasets.

The filenames, key names, object/array structure, and value types are compatibility requirements. A consumer joins the files using the provider key and selects a model by its `name` within that provider. Do not merge entries across providers solely because they refer to the same underlying model.

No schema file, parser implementation, API client, synchronization job, or credential store is supplied. Integration behaviour described below is the responsibility of the consuming application.

## Module map

| Module | Path | Responsibility | Depends on | Used by |
| --- | --- | --- | --- | --- |
| Providers | `api-providers.json` | Connection metadata and provider-specific hints | Provider documentation | External provider adapters and selectors |
| Models | `api-models.json` | Model identifiers and metadata by provider | Shared provider keys and model sources | External model selectors and request configuration |
| Overview | `README.md`, `doc/README.*.md` | Purpose, Debian setup, and quick start | Both catalogs | Users and integrators |
| Manual | `doc/MANUAL*.md` | Operational examples and troubleshooting | Both catalogs | Users |
| Technical reference | `doc/CODE*.md` | Data contract and integration responsibilities | Both catalogs | Developers and coding assistants |

## Key data symbols

There are no code symbols to index. These JSON keys are the main navigation points; line numbers refer to the files when this documentation was created and may move as entries change.

| Data symbol | File:line | Role |
| --- | --- | --- |
| Provider root object | `api-providers.json:1` | Provider-key lookup |
| Model root object | `api-models.json:1` | Provider-to-model-array lookup |
| `anthropic` | `api-providers.json:2`, `api-models.json:2` | Native request-format example |
| `llamacpp` | `api-providers.json:35`, `api-models.json:625` | Local-service example |
| `openai` | `api-providers.json:46`, `api-models.json:1133` | Primary and Responses URL example |
| `vercel` | `api-providers.json:151`, `api-models.json:7204` | Nested reasoning-format hint |
| `cloudflare` | `api-providers.json:159`, `api-models.json:9054` | Account placeholder and credential-format hint |
| `freetoken` | `api-providers.json:165`, `api-models.json:9182` | Configured provider with an empty model array |

## Provider fields

All provider field values currently use strings. `name`, `url`, and `format` occur in every provider object; the other fields are optional.

| Field | Meaning |
| --- | --- |
| `name` | Display name. The enclosing JSON key is the provider identifier. |
| `url` | Primary API URL. Some providers supply a full operation endpoint; Gemini supplies a base URL. |
| `format` | Adapter label. Recorded values are `anthropic`, `cohere`, `gemini`, `llamacpp`, and `openai`. |
| `url_responses` | Alternative endpoint for models marked `api_type: "responses"`. |
| `max_tokens_field` | Output-limit parameter name, such as `max_tokens` or `max_completion_tokens`. |
| `tool_choice_mode` | Tool-choice hint. Recorded values are `auto`, `any`, and `omit`; the adapter must implement their provider-specific handling. |
| `reasoning_format` | Reasoning serialisation hint; `nested` is recorded for Vercel. |
| `credential_format` | Credential layout hint; Cloudflare records `account_id:api_token`, with `{account_id}` in its URL. |

Absence of an optional field does not define a universal fallback. The appropriate adapter must supply behaviour consistent with its provider. Credentials are supplied separately, and credential-format metadata must not be sent verbatim as an authorization token.

## Model fields

All ordinary model fields below are strings, including numbers and boolean-like flags. The optional `reasoning_effort_map` is an object whose recorded values are integers.

| Field | Meaning |
| --- | --- |
| `name` | Exact model identifier used within the selected provider. |
| `creation_date` | Recorded date in `yYYYYmMMdDD` form, for example `y2026m10d19`, or an empty string. Provider listing dates may differ from the original model release date. |
| `tokens_max_context_window` | Recorded context-window size in tokens. |
| `tokens_max_output` | Recorded output-token limit. |
| `price_per_million_tokens_input` | Reference input price in USD per million tokens. |
| `price_per_million_tokens_cached_input` | Reference cached-input price in USD per million tokens. |
| `price_per_million_tokens_output` | Reference output price in USD per million tokens. |
| `model_info_url` | Source for model details. |
| `provider_models_url` | Source for the provider's model catalogue or documentation. |
| `is_mixture_of_experts` | Recorded MoE architecture flag: `"true"`, `"false"`, or empty. |
| `max_chars_per_request` | Recorded character limit, independent of token counts. |
| `reasoning` | String flag describing reasoning support. |
| `tool_calling` | String flag describing tool-calling support; it does not identify a particular hosted tool or search service. |
| `accepts_temperature` | String flag describing support for a configurable temperature. |
| `reasoning_mode` | Catalogue configuration label; see the modes below. |
| `reasoning_effort_levels` | Comma-separated effort labels, not a JSON array. |
| `reasoning_effort_default` | Recorded default effort label. |
| `reasoning_effort_source` | Source marker; currently `docs` or empty. |
| `api_type` | Operation-family label: `completions` or `responses`. Interpret together with the provider's `format`. |
| `reasoning_effort_map` | Optional mapping from effort labels to numeric token budgets. |

The recorded reasoning-mode labels are `adaptive`, `budget`, `effort`, `effort_toggle`, `toggle`, and the empty string. They describe configuration styles for a consumer to interpret; they are not a universal API payload. For `budget` entries, an optional map supplies values such as `"low": 1024`. Preserve these values as numbers inside the map.

An empty string means no value is recorded. Do not convert it to `0`, `false`, an unlimited limit, or an inferred default. Similarly, parse `"false"` explicitly instead of using the truthiness of a nonempty string. Convert numeric strings only in the consumer's working data, without changing the source representation.

Prices describe one recorded tariff per field. Tiered context pricing, route selection, promotion dates, fast or batch modes, cache storage, and per-request or audio charges are not represented. An entry's price does not encode its access conditions.

## Main flows

These are integration flows, not call paths implemented in this repository.

1. **Browse:** read providers → select provider key → read the matching model array → display model names and metadata.
2. **Configure a request:** select provider/model pair → interpret `format` and `api_type` → choose or construct the URL → apply supported options → supply credentials → send through an external client.
3. **Update data:** review provider sources → edit the affected catalogue entries → preserve the data contract → distribute both files together → refresh consumers and review existing selections.
4. **Retire a model:** confirm the affected provider and identifier → remove the entry upon announcement → retain other providers' entries unless separately affected → select a replacement in consumers when needed.

## Entry points

| Path or command | Handler | File |
| --- | --- | --- |
| Read `api-providers.json` | Consuming application's JSON parser | `api-providers.json` |
| Read `api-models.json` | Consuming application's JSON parser | `api-models.json` |
| `jq -r 'keys[]' api-providers.json` | External `jq` utility | `api-providers.json` |
| `jq -r '.openai[].name' api-models.json` | External `jq` utility | `api-models.json` |

The repository exposes no HTTP routes or application CLI. URLs inside the provider catalogue describe external or local services; this repository does not start them.

## Impact analysis

| Change | Affected consumers |
| --- | --- |
| Rename a file, key, field, or change a value type | Parsers, joins, stored selections, and configuration loaders; incompatible with the current contract |
| Change a provider URL or integration hint | Every model request using that provider's adapter |
| Add or remove a model | Model selectors and saved model choices |
| Change prices or limits | Displays, estimates, and request-sizing logic |
| Change reasoning, tools, or temperature metadata | Request builders and configuration controls |
| Change a source URL | Maintenance and traceability |

## Extension points

To add a provider, create its configuration object in `api-providers.json` and a matching array in `api-models.json`; the array may be empty. Populate only supported integration hints. Any new adapter behaviour must be implemented in the consuming applications.

To add a model, use the exact identifier for that provider and preserve the existing model structure. Populate documented values, keep unrecorded string values empty, and include source links. Add `reasoning_effort_map` only when the numeric mapping is supported.

To revise an entry, update the affected provider's data without propagating unconfirmed assumptions to other services. Remove announced deprecated or retired identifiers immediately. Features that cannot be represented by the existing schema require separate design work; do not disguise them as chat models or token prices.

Keep this reference, its translations, the README, and the manual aligned with structural changes. Update relevant module, symbol, and flow entries when their locations or behaviour change.
