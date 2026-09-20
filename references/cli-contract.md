# GPTProto CLI contract

The CLI has four public responsibilities: discover models and their call formats, query live public pricing, execute a native request body, and extract a final result.

## Commands

```bash
gptproto config
gptproto key set --key KEY
gptproto key remove
gptproto --version
gptproto version [--json]
gptproto update [VERSION]

gptproto models list [--capability text|image|video|audio] [--json]
gptproto model <provider/model> [--json]

gptproto pricing list [--capability text|image|video|audio] [--mode TAG] [--search TEXT] [--sort price|name] [--limit N] [--json]
gptproto pricing <text|image|images|video|videos|audio> [--mode TAG] [--cheapest N] [--json]
gptproto pricing <provider/model> [--json]

gptproto request <METHOD> <PATH> --json '<native JSON object>'
gptproto request <METHOD> <PATH> --body request.json
gptproto request <METHOD> <PATH> --form name=value --file field=local-file

gptproto custom create <resource> --json '<GPTProto custom JSON object>' --wait
gptproto task get <TASK_ID> [--video]
gptproto task wait <TASK_ID> [--video]
```

`text`, `image`, `video`, `audio`, `call`, and `generate` are intentionally not public commands in this contract. `gptproto key set --key KEY` is a command for the user to run in their own terminal; the Skill must not receive the real key in chat or execute that command with a real key supplied through chat. `gptproto version` queries published versions from npm; `gptproto update` installs npm's latest release, and a version argument installs that exact published version. The user must request an update before the Skill runs it.

## Pricing

`gptproto pricing` reads GPTProto's live public model catalog and does not require an API key. Use an exact `provider/model` to inspect one model, a capability shorthand to compare one output category, or `pricing list` for broader filtering. `--mode` matches the catalog's exact `modelTag` value, such as `text-to-image` or `image-to-video`; it is not forwarded to a generation endpoint. `--search` searches catalog model identifiers and names. `--cheapest N` is a convenience for price sorting plus a result limit.

JSON output contains the model identity, capability and tags, normalized rates, billing unit, and `starting_price`. A low `starting_price` is useful for ordering candidates but is not an exact cost estimate. Do not compare unlike billing units as though they were equivalent, and do not treat catalog presence as evidence of route compatibility or channel availability. After price-based selection, validate the candidate with `gptproto model <provider/model> --json` before constructing a request.

## Request inputs

`--json` is an inline JSON request body. `--body` is a JSON file. `--form` and `--file` are for registered multipart routes only. `--stream` requests streaming result handling; the provider's required stream field remains part of the native JSON body.

The CLI does not synthesize `input`, `messages`, or `contents`, or translate provider-specific options. It does perform one model-value normalization: for official-compatible JSON or multipart requests, a catalog model ID such as `openai/gpt-4.1` is sent as `gpt-4.1`; custom GPTProto routes retain `openai/gpt-4.1`. The Skill obtains all other details from the selected live model descriptor and sends the exact documented shape. There is no public global endpoint registry: `gptproto model <provider/model>` is the discovery command.

## Response outputs

The default output is extracted by the CLI:

| Interface response | Default CLI output |
| --- | --- |
| OpenAI Responses, Chat Completions, Claude Messages, Gemini | text |
| Documented image, video, and async media interfaces | final URL(s) |
| Binary response | requires `--output FILE` |
| Other JSON response | original JSON |

`--output-json` prints the unprocessed response JSON. `--raw` with `--stream` prints raw SSE events. The Skill should not implement a second parser for provider response objects.

## Async behavior

`gptproto custom create ... --wait` and `gptproto request ... --wait` poll documented asynchronous tasks. The default HTTP timeout and maximum polling duration are 600 seconds. On timeout, do not repeat a billable generation request; query the task ID instead.
