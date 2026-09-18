# GPTProto Skill

An installable Skill that lets people ask for text, images, video, or audio in ordinary language. The AI discovers GPTProto model interfaces, builds native requests, runs GPTProto CLI, and returns the result.

It does not maintain a model list, map one provider's schema to another, or contain a duplicate provider SDK. GPTProto and the GPTProto SDK remain the interface-contract source of truth.

## Install

For local development:

```bash
npx skills add /path/to/gptproto-skill --skill gptproto-skill --global --agent codex --yes
```

From the public GitHub repository:

```bash
npx skills add gptproto-ai/gptproto-skill --skill gptproto-skill --global --agent codex --yes
```

Start a new Codex conversation after installation.

## Prerequisites

The Skill and the CLI are separate installations. Install GPTProto CLI first, then set the API Key in **your own terminal**, never in an AI chat; replace the placeholder locally:

```bash
npm install -g @gptproto-ai/cli
gptproto --version
gptproto key set --key YOUR_GPTPROTO_API_KEY
gptproto config
```

The Skill only checks whether a key is configured. It never asks you to paste the key into chat or changes the selected server unless you ask. On a local Mac, it can open Terminal for you to run the setup command; if that is unavailable, it tells you exactly what to run.

You can then ask naturally: "Make a picture of a cat," "Summarize this document," or "Which GPTProto CLI version am I using?" The Skill handles model discovery and native request details. If you explicitly ask to update the CLI, it uses `gptproto update` or `gptproto update <version>` after checking npm's published versions.

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
