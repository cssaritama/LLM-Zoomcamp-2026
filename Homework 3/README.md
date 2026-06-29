# LLM Zoomcamp 2026 — Homework 3: AI Orchestration with Kestra

My solution for Module 3. I set up Kestra locally with the Gemini API key, imported the example flows from `03-orchestration/flows/`, ran them from the Kestra UI, and answered the questions from the execution logs.

## Setup

- Kestra running locally via Docker, with `GEMINI_API_KEY` stored as a secret.
- All flows from `03-orchestration/flows/` imported into the `zoomcamp` namespace.
- Agents run on Google Gemini (`gemini-2.5-flash`), as defined in the flow's `pluginDefaults`.

## Answers

**Q1 — Context engineering: why AI Copilot writes better Kestra flows**
> AI Copilot has access to current Kestra plugin documentation.

I tried the same prompt ("Create a Kestra flow that loads NYC taxi data from CSV to BigQuery") in ChatGPT and in Kestra's AI Copilot. ChatGPT produced YAML that looks right but invents plugin types and properties that don't exist. Copilot is grounded in Kestra's current plugin docs, so its flow actually uses valid, up-to-date plugin syntax.

**Q2 — RAG vs no RAG**
> Vague, generic, or fabricated — the model guesses from training data.

`1_chat_without_rag.yaml` answers about Kestra 1.1 from training data, which is older than that release, so the response is generic and partly made up. `2_chat_with_rag.yaml`, grounded in the indexed docs, returns specific, correct features.

**Q3 — Token usage, short summary**
> 60-100 output tokens.

With `summary_length = short` the system prompt asks for a 1-2 sentence summary, and `multilingual_agent` logs around 80 output tokens.

**Q4 — Token usage, long summary**
> 2-5x more.

Switching to `summary_length = long` (1-3 paragraphs) pushes `multilingual_agent` output to roughly 300 tokens — about 3.7x the short run.

**Q5 — Modifying the flow**
> 2-4x more.

After changing `english_brevity` from "exactly 1 sentence" to "exactly 3 sentences" (both with `summary_length = long`), its output went from ~29 to ~81 tokens — about 2.8x more. Output scales roughly with the number of sentences requested.

**Q6 — Best practices for regulated, deterministic workflows**
> Use traditional task-based workflows for predictability and auditability.

Agents are useful when you need flexibility, but they're non-deterministic. For compliance-heavy or financial workflows you want runs that are repeatable and auditable, which a plain task-based flow gives you.

## Token usage (from the `log_token_usage` task)

Run A — `summary_length = short` (other inputs left as defaults):

| Task | Input | Output | Total |
| --- | --- | --- | --- |
| `multilingual_agent` | 372 | 84 | 456 |
| `english_brevity` | 108 | 27 | 135 |

Run B — `summary_length = long`:

| Task | Input | Output | Total |
| --- | --- | --- | --- |
| `multilingual_agent` | 372 | 309 | 681 |
| `english_brevity` (1 sentence) | 333 | 29 | 362 |

Run C — `summary_length = long`, `english_brevity` changed to 3 sentences:

| Task | Input | Output | Total |
| --- | --- | --- | --- |
| `multilingual_agent` | 372 | 309 | 681 |
| `english_brevity` (3 sentences) | 333 | 81 | 414 |

Output tokens: `multilingual_agent` short → long is 84 → 309 (~3.7x). `english_brevity` 1 → 3 sentences is 29 → 81 (~2.8x).

## Change made for Q5

In `flows/4_simple_agent.yaml`, the `english_brevity` prompt:

```yaml
prompt: |
  Generate exactly 3 sentences English summary of the following:
  "{{ outputs.multilingual_agent.textOutput }}"
```

It originally asked for exactly 1 sentence.

## What I took away

The pattern across the whole module is that context, not the model, is what moves the needle: the same Gemini model gives a far better answer once it can see the right plugin docs or grounded data. Token usage tracks the length you ask for almost linearly, so prompt wording has a direct, measurable effect on cost. And when a workflow has to be predictable and auditable, a task-based flow is the safer choice over an agent.
