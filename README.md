# Unsiloed Skill for OpenClaw

Vision LLMs hallucinate on hard documents. Hand a budget-tier model a doctor's prescription and it confidently returns medicines that aren't on the page — no flag, no confidence score, no way for downstream code to know something went wrong.

This skill swaps that read for a call to [Unsiloed](https://www.unsiloed.ai)'s document AI API. With it installed, any document attachment routes through Unsiloed instead of the agent's vision. The agent gets back structured data with per-field confidence scores, so the reply can flag what to verify.

Everything lives in a single `SKILL.md` file using curl and jq. No Python, no extra runtimes.

## What the Skill Covers

The skill teaches the agent four Unsiloed operations and how to pick between them:

- **Parse** — read everything on the page and return Markdown of each layout region. The default for "what does this say"
- **Extract** — pull named fields out as JSON with a confidence score per field. Use this when the user asks for structured output
- **Classify** — label a document as one of several candidate categories
- **Split** — break a single PDF that contains several documents into separate files by category

## Installing the Skill

Install directly from this repo using the OpenClaw CLI:

```
openclaw skills install git:https://github.com/LewisDwyer/unsiloed-skill
```

OpenClaw fetches the repo, registers the skill in the workspace, and tracks the origin for future updates with `openclaw skills update --all`.

## Setting the Unsiloed API Key

The skill needs `UNSILOED_API_KEY` in the gateway's environment. Get a key from [unsiloed.ai](https://www.unsiloed.ai), then append it to the OpenClaw global env file:

```
echo 'UNSILOED_API_KEY=us_...' >> ~/.openclaw/.env
chmod 600 ~/.openclaw/.env
```

Restart the gateway so the new env var loads:

```
openclaw gateway restart
```

Verify the skill is ready:

```
openclaw skills info unsiloed
```

The first line of the output should read `unsiloed ✓ Ready`. To see it alongside every other skill, run `openclaw skills check` — `unsiloed` should appear under **Ready and visible to model**.

## Using the Skill

Send a document to the OpenClaw agent through any connected channel (Telegram, file share, paired terminal). The skill auto-invokes whenever a document attachment or document-shaped question appears. Example:

> **You:** [attached prescription.jpg] What medicines are listed here?
>
> **Agent:** Three medicines: Tab Azee 500mg, Tab Montair FX, Tab Dolo 650. All extracted with confidence above 0.95.

The agent reads the SKILL.md, picks the right Unsiloed operation, builds the curl call, polls until the async job completes, and replies with the extracted content. The user never sees raw JSON.

## Links

- [OpenClaw skill format and CLI](https://docs.openclaw.ai/cli/skills)
- [Unsiloed API reference](https://docs.unsiloed.ai)
- [Get an Unsiloed API key](https://www.unsiloed.ai)
