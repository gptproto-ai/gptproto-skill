# GPTProto Skill

An installable Skill that lets an AI agent discover GPTProto model interfaces, build native request JSON, run GPTProto CLI, and return extracted results.

It does not maintain a model list, map one provider's schema to another, or contain a duplicate provider SDK. GPTProto and the GPTProto SDK remain the interface-contract source of truth.

## Install

For local development:

```bash
npx skills add /path/to/gptproto-skill --skill gptproto-skill --global --agent codex --yes
```

After publishing the repository as `gptproto-ai/gptproto-skill`:

```bash
npx skills add gptproto-ai/gptproto-skill --skill gptproto-skill --global --agent codex --yes
```

Start a new Codex conversation after installation.

## Prerequisites

Install and configure GPTProto CLI independently:

```bash
gptproto --version
gptproto key set --key YOUR_API_KEY
gptproto config
```

The Skill never stores a key or changes the selected server unless the user asks.

## Runtime flow

```text
user request
  -> gptproto models list
  -> gptproto model <provider/model>
  -> native JSON request body
  -> gptproto request or gptproto custom create
  -> CLI extracts text, URL, or file result
```

Examples of the native CLI execution that the Skill produces:

```bash
gptproto request POST /v1/responses \
  --json '{"model":"openai/gpt-4.1","input":"Hello"}'

gptproto request POST /v1/messages \
  --json '{"model":"claude/claude-opus-4-7","max_tokens":1024,"messages":[{"role":"user","content":"Hello"}]}'

gptproto custom create image \
  --json '{"model":"openai/gpt-image-2","prompt":"A tiger"}' --wait
```

See [SKILL.md](SKILL.md) for agent instructions and [references/cli-contract.md](references/cli-contract.md) for the public CLI contract.
