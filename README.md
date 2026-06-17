# Agent Storming

**Agent Storming** is a collaborative modelling workshop for designing business processes executed by AI agents. It reinterprets Alberto Brandolini's **Event Storming** — same wall, same left-to-right timeline, same coloured-sticky grammar — but changes the subject from *what happens in a domain* to *which agents do what, when, and with what guardrails and human checkpoints*.

This repository is the **method**, not an app. It documents how to run Agent Storming, how it differs from Event Storming, and how to adopt it in your business.

## 📖 Read the docs

**→ [dholliday.github.io/agent-storming](https://dholliday.github.io/agent-storming/)** — the full guide, beautifully hosted (search, dark mode, printable cheat sheet).

The site is built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) from the Markdown in [`docs/`](./docs) and deployed to GitHub Pages automatically on every push to `main` (see [`.github/workflows/docs.yml`](./.github/workflows/docs.yml)).

### Preview locally

```bash
python3 -m venv .venv-docs && source .venv-docs/bin/activate
pip install -r requirements-docs.txt
mkdocs serve   # http://127.0.0.1:8000
```

## Start here

📖 **[The comprehensive guide](https://dholliday.github.io/agent-storming/guide/what-is-agent-storming/)** covers:

- What Agent Storming is and why it exists
- How it differs from Event Storming (element mapping + agent-native additions)
- The full grammar / notation reference
- The three workshop levels
- How to facilitate a session, step by step
- A worked end-to-end example (AI support-ticket triage)
- How to implement it in your business
- A printable cheat sheet

## Related

A companion Miro-style web app that supports the method (an infinite collaborative canvas with the grammar built in) is specified separately in the [companion app spec](https://dholliday.github.io/agent-storming/app-spec/) ([source](./docs/app-spec.md)). You don't need the app to run a session — a wall and sticky notes are enough.
